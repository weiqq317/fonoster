# Fonoster 部署和配置指南

## 概述

本文档提供了 Fonoster 平台的完整部署和配置指南，包括开发环境、生产环境和云环境的部署方案。

## 目录

- [系统要求](#系统要求)
- [Docker Compose 部署](#docker-compose-部署)
- [Kubernetes 部署](#kubernetes-部署)
- [环境变量配置](#环境变量配置)
- [网络配置](#网络配置)
- [安全配置](#安全配置)
- [监控和日志](#监控和日志)
- [备份和恢复](#备份和恢复)
- [故障排除](#故障排除)

## 系统要求

### 最低硬件要求

#### 开发环境
- **CPU**: 2 核心
- **内存**: 4GB RAM
- **存储**: 20GB 可用空间
- **网络**: 稳定的互联网连接

#### 生产环境
- **CPU**: 4 核心 (推荐 8 核心)
- **内存**: 8GB RAM (推荐 16GB)
- **存储**: 100GB SSD (推荐 500GB)
- **网络**: 高速网络连接，低延迟

### 软件要求

- **Docker**: 20.10+ 
- **Docker Compose**: 2.0+
- **Node.js**: 18+ (仅开发环境)
- **PostgreSQL**: 13+ (如果使用外部数据库)
- **操作系统**: Linux (推荐 Ubuntu 20.04+), macOS, Windows

### 网络端口

| 服务 | 端口 | 协议 | 描述 |
|------|------|------|------|
| Dashboard | 3030 | HTTP | Web 管理界面 |
| API Server | 50051 | gRPC | API 服务器 |
| Autopilot | 50061 | gRPC | 智能助手服务 |
| Routr | 51907, 51908 | HTTP/gRPC | SIP 路由器 |
| Routr SIP | 5060-5063 | SIP/UDP | SIP 信令 |
| Asterisk | 6060 | SIP | Asterisk PBX |
| RTPEngine | 10000-20000 | RTP/UDP | 媒体流 |
| PostgreSQL | 5432 | TCP | 数据库 |
| InfluxDB | 8086 | HTTP | 时序数据库 |
| NATS | 4222 | TCP | 消息队列 |
| Envoy | 8449 | HTTP | 反向代理 |

## Docker Compose 部署

### 快速开始

1. **克隆项目**
```bash
git clone https://github.com/fonoster/fonoster.git
cd fonoster
```

2. **配置环境变量**
```bash
# 复制环境变量模板
cp .env.example .env

# 编辑配置文件
nano .env
```

3. **生成密钥对**
```bash
# 创建密钥目录
mkdir -p config/keys

# 生成 RSA 密钥对
openssl genrsa -out config/keys/private.pem 2048
openssl rsa -in config/keys/private.pem -pubout -out config/keys/public.pem
```

4. **启动服务**
```bash
# 启动所有服务
docker compose up -d

# 查看服务状态
docker compose ps

# 查看日志
docker compose logs -f
```

### 开发环境部署

开发环境使用 `compose.dev.yaml` 配置文件，提供了额外的开发工具和调试功能。

```bash
# 启动开发环境
docker compose -f compose.yaml -f compose.dev.yaml up -d

# 包含额外服务
# - Adminer: 数据库管理工具 (端口 8282)
# - MailHog: 邮件测试工具 (端口 8025)
```

### 生产环境部署

生产环境部署需要额外的安全和性能配置：

```bash
# 设置生产环境变量
export NODE_ENV=production

# 使用生产配置启动
docker compose up -d

# 启用 HTTPS (可选)
# 需要配置 SSL 证书和 Envoy 代理
```

## Kubernetes 部署

### 准备工作

1. **创建命名空间**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: fonoster
```

2. **创建配置映射**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fonoster-config
  namespace: fonoster
data:
  # 从 .env 文件导入配置
```

3. **创建密钥**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: fonoster-secrets
  namespace: fonoster
type: Opaque
data:
  # Base64 编码的敏感配置
```

### 部署清单示例

#### PostgreSQL 部署
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: fonoster
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:16.10-alpine3.22
        env:
        - name: POSTGRES_USER
          value: "postgres"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: fonoster-secrets
              key: postgres-password
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  namespace: fonoster
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
```

#### API Server 部署
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apiserver
  namespace: fonoster
spec:
  replicas: 2
  selector:
    matchLabels:
      app: apiserver
  template:
    metadata:
      labels:
        app: apiserver
    spec:
      containers:
      - name: apiserver
        image: fonoster/apiserver:0.15.15
        envFrom:
        - configMapRef:
            name: fonoster-config
        - secretRef:
            name: fonoster-secrets
        ports:
        - containerPort: 50051
        volumeMounts:
        - name: keys-volume
          mountPath: /opt/fonoster/keys
          readOnly: true
      volumes:
      - name: keys-volume
        secret:
          secretName: fonoster-keys
---
apiVersion: v1
kind: Service
metadata:
  name: apiserver
  namespace: fonoster
spec:
  selector:
    app: apiserver
  ports:
  - port: 50051
    targetPort: 50051
```

### Helm Chart 部署

创建 Helm Chart 用于简化 Kubernetes 部署：

```bash
# 创建 Helm Chart
helm create fonoster

# 安装 Chart
helm install fonoster ./fonoster \
  --namespace fonoster \
  --create-namespace \
  --values values.yaml
```

## 环境变量配置

### 核心配置

#### API Server 配置
```bash
# 应用 URL
APISERVER_APP_URL=https://your-domain.com

# 数据库连接
APISERVER_DATABASE_URL=postgresql://user:password@host:5432/fonoster
APISERVER_IDENTITY_DATABASE_URL=postgresql://user:password@host:5432/fnidentity

# 加密密钥 (使用 openssl rand -base64 32 生成)
APISERVER_CLOAK_ENCRYPTION_KEY=your-encryption-key

# SMTP 配置
APISERVER_SMTP_HOST=smtp.your-domain.com
APISERVER_SMTP_PORT=587
APISERVER_SMTP_SECURE=true
APISERVER_SMTP_AUTH_USER=noreply@your-domain.com
APISERVER_SMTP_AUTH_PASS=your-smtp-password
APISERVER_SMTP_SENDER="Your App <noreply@your-domain.com>"

# 管理员账户
APISERVER_OWNER_EMAIL=admin@your-domain.com
APISERVER_OWNER_NAME=Administrator
APISERVER_OWNER_PASSWORD=secure-password
```

#### Asterisk 配置
```bash
# ARI 接口
APISERVER_ASTERISK_ARI_PROXY_URL=http://asterisk:8088
APISERVER_ASTERISK_ARI_USERNAME=ari
APISERVER_ASTERISK_ARI_SECRET=secure-ari-secret

# SIP 配置
ASTERISK_SIPPROXY_HOST=your-public-ip
ASTERISK_SIPPROXY_PORT=5060
ASTERISK_SIPPROXY_USERNAME=voice
ASTERISK_SIPPROXY_SECRET=secure-sip-secret

# RTP 端口范围
ASTERISK_RTP_PORT_START=10000
ASTERISK_RTP_PORT_END=20000

# 编解码器
ASTERISK_CODECS=g722,ulaw,alaw
ASTERISK_DTMF_MODE=auto_info
```

#### Routr 配置
```bash
# 数据库
ROUTR_DATABASE_URL=postgresql://user:password@host:5432/routr

# 外部地址 (必须设置为公网 IP)
ROUTR_EXTERNAL_ADDRS=your-public-ip

# RTPEngine
ROUTR_RTPENGINE_HOST=rtpengine

# NATS 消息队列
ROUTR_NATS_PUBLISHER_ENABLED=true
ROUTR_NATS_PUBLISHER_URL=nats://nats:4222
```

#### RTPEngine 配置
```bash
# 公网 IP (必须设置)
RTPENGINE_PUBLIC_IP=your-public-ip

# 端口范围
RTPENGINE_PORT_MIN=10000
RTPENGINE_PORT_MAX=20000
```

### 可选配置

#### OAuth2 认证
```bash
# GitHub OAuth2
APISERVER_IDENTITY_OAUTH2_GITHUB_ENABLED=true
APISERVER_IDENTITY_OAUTH2_GITHUB_CLIENT_ID=your-github-client-id
APISERVER_IDENTITY_OAUTH2_GITHUB_CLIENT_SECRET=your-github-client-secret
```

#### Twilio 集成
```bash
APISERVER_TWILIO_ACCOUNT_SID=your-twilio-account-sid
APISERVER_TWILIO_AUTH_TOKEN=your-twilio-auth-token
APISERVER_TWILIO_PHONE_NUMBER=your-twilio-phone-number
```

#### Autopilot AI 功能
```bash
# OpenAI 集成
AUTOPILOT_OPENAI_API_KEY=your-openai-api-key

# 知识库功能 (需要 S3 兼容存储)
AUTOPILOT_KNOWLEDGE_BASE_ENABLED=true
AUTOPILOT_AWS_S3_ACCESS_KEY_ID=your-s3-access-key
AUTOPILOT_AWS_S3_SECRET_ACCESS_KEY=your-s3-secret-key
AUTOPILOT_AWS_S3_ENDPOINT=https://s3.amazonaws.com
AUTOPILOT_AWS_S3_REGION=us-east-1

# Unstructured API (文档处理)
AUTOPILOT_UNSTRUCTURED_API_KEY=your-unstructured-api-key
AUTOPILOT_UNSTRUCTURED_API_URL=https://api.unstructured.io
```

## 网络配置

### 防火墙设置

#### 入站规则
```bash
# HTTP/HTTPS 流量
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8449/tcp  # Envoy 代理

# SIP 信令
sudo ufw allow 5060:5063/udp
sudo ufw allow 5060:5063/tcp

# RTP 媒体流
sudo ufw allow 10000:20000/udp

# 管理端口 (仅限内网)
sudo ufw allow from 10.0.0.0/8 to any port 3030    # Dashboard
sudo ufw allow from 10.0.0.0/8 to any port 50051   # API Server
```

#### 出站规则
```bash
# 允许所有出站流量 (默认)
sudo ufw default allow outgoing

# 或者限制特定端口
sudo ufw allow out 53      # DNS
sudo ufw allow out 80      # HTTP
sudo ufw allow out 443     # HTTPS
sudo ufw allow out 587     # SMTP
sudo ufw allow out 5432    # PostgreSQL
```

### NAT 配置

对于云环境或 NAT 后的部署，需要正确配置网络地址转换：

```bash
# 设置公网 IP
export PUBLIC_IP=$(curl -s ifconfig.me)

# 更新环境变量
sed -i "s/ROUTR_EXTERNAL_ADDRS=.*/ROUTR_EXTERNAL_ADDRS=${PUBLIC_IP}/" .env
sed -i "s/ASTERISK_SIPPROXY_HOST=.*/ASTERISK_SIPPROXY_HOST=${PUBLIC_IP}/" .env
sed -i "s/RTPENGINE_PUBLIC_IP=.*/RTPENGINE_PUBLIC_IP=${PUBLIC_IP}/" .env
```

### 负载均衡

#### Nginx 配置示例
```nginx
upstream fonoster_api {
    server 127.0.0.1:50051;
    server 127.0.0.1:50052;  # 如果有多个实例
}

upstream fonoster_dashboard {
    server 127.0.0.1:3030;
    server 127.0.0.1:3031;  # 如果有多个实例
}

server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    location / {
        proxy_pass http://fonoster_dashboard;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        grpc_pass grpc://fonoster_api;
        grpc_set_header Host $host;
    }
}
```

## 安全配置

### SSL/TLS 配置

#### Let's Encrypt 证书
```bash
# 安装 Certbot
sudo apt install certbot

# 获取证书
sudo certbot certonly --standalone -d your-domain.com

# 配置自动续期
sudo crontab -e
# 添加: 0 12 * * * /usr/bin/certbot renew --quiet
```

#### 自签名证书 (仅开发环境)
```bash
# 生成自签名证书
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

### 数据库安全

#### PostgreSQL 安全配置
```sql
-- 创建专用用户
CREATE USER fonoster_api WITH PASSWORD 'secure_password';
CREATE USER fonoster_identity WITH PASSWORD 'secure_password';

-- 创建数据库
CREATE DATABASE fonoster OWNER fonoster_api;
CREATE DATABASE fnidentity OWNER fonoster_identity;
CREATE DATABASE routr OWNER fonoster_api;

-- 设置权限
GRANT ALL PRIVILEGES ON DATABASE fonoster TO fonoster_api;
GRANT ALL PRIVILEGES ON DATABASE fnidentity TO fonoster_identity;
GRANT ALL PRIVILEGES ON DATABASE routr TO fonoster_api;
```

#### 数据库连接加密
```bash
# 启用 SSL 连接
APISERVER_DATABASE_URL=postgresql://user:password@host:5432/fonoster?sslmode=require
```

### 访问控制

#### API 密钥管理
```bash
# 使用 CLI 创建 API 密钥
fonoster apikeys:create --name "Production API Key"

# 设置权限范围
fonoster apikeys:update <key-id> --scopes "calls:read,calls:write"
```

#### 工作空间隔离
```bash
# 创建工作空间
fonoster workspaces:create --name "Production Workspace"

# 邀请用户
fonoster workspaces:invite --email user@company.com --role member
```

### 网络安全

#### 服务间通信加密
```yaml
# 在 Docker Compose 中启用 TLS
services:
  apiserver:
    environment:
      - TLS_ENABLED=true
      - TLS_CERT_PATH=/opt/fonoster/certs/server.crt
      - TLS_KEY_PATH=/opt/fonoster/certs/server.key
    volumes:
      - ./certs:/opt/fonoster/certs:ro
```

## 监控和日志

### 日志配置

#### 集中化日志
```bash
# 配置日志格式
APISERVER_LOGS_FORMAT=json
APISERVER_LOGS_LEVEL=info
APISERVER_LOGS_TRANSPORT=file

# 使用 ELK Stack
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  elasticsearch:7.14.0

docker run -d \
  --name kibana \
  -p 5601:5601 \
  --link elasticsearch:elasticsearch \
  kibana:7.14.0
```

#### 日志轮转
```bash
# 配置 logrotate
sudo nano /etc/logrotate.d/fonoster

# 内容:
/var/log/fonoster/*.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    create 644 fonoster fonoster
    postrotate
        docker kill -s USR1 $(docker ps -q --filter name=fonoster)
    endscript
}
```

### 性能监控

#### Prometheus 配置
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'fonoster-api'
    static_configs:
      - targets: ['localhost:50051']
    metrics_path: /metrics

  - job_name: 'fonoster-asterisk'
    static_configs:
      - targets: ['localhost:6060']
```

#### Grafana 仪表板
```bash
# 启动 Grafana
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin" \
  grafana/grafana

# 导入预配置的仪表板
# 访问 http://localhost:3000
# 导入 Fonoster 仪表板模板
```

### 健康检查

#### Docker 健康检查
```yaml
services:
  apiserver:
    healthcheck:
      test: ["CMD", "grpc_health_probe", "-addr=:50051"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

#### Kubernetes 探针
```yaml
spec:
  containers:
  - name: apiserver
    livenessProbe:
      exec:
        command:
        - grpc_health_probe
        - -addr=:50051
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      exec:
        command:
        - grpc_health_probe
        - -addr=:50051
      initialDelaySeconds: 5
      periodSeconds: 5
```

## 备份和恢复

### 数据库备份

#### 自动备份脚本
```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/fonoster"
DATE=$(date +%Y%m%d_%H%M%S)

# 创建备份目录
mkdir -p $BACKUP_DIR

# 备份 PostgreSQL 数据库
docker exec postgres pg_dump -U postgres fonoster > $BACKUP_DIR/fonoster_$DATE.sql
docker exec postgres pg_dump -U postgres fnidentity > $BACKUP_DIR/fnidentity_$DATE.sql
docker exec postgres pg_dump -U postgres routr > $BACKUP_DIR/routr_$DATE.sql

# 备份 InfluxDB
docker exec influxdb influx backup /tmp/backup_$DATE
docker cp influxdb:/tmp/backup_$DATE $BACKUP_DIR/influxdb_$DATE

# 压缩备份
tar -czf $BACKUP_DIR/fonoster_backup_$DATE.tar.gz $BACKUP_DIR/*_$DATE.*

# 清理旧备份 (保留 30 天)
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: fonoster_backup_$DATE.tar.gz"
```

#### 定时备份
```bash
# 添加到 crontab
crontab -e

# 每天凌晨 2 点备份
0 2 * * * /path/to/backup.sh
```

### 数据恢复

#### PostgreSQL 恢复
```bash
# 停止服务
docker compose stop apiserver

# 恢复数据库
docker exec -i postgres psql -U postgres -c "DROP DATABASE IF EXISTS fonoster;"
docker exec -i postgres psql -U postgres -c "CREATE DATABASE fonoster;"
docker exec -i postgres psql -U postgres fonoster < backup/fonoster_20240101_020000.sql

# 重启服务
docker compose start apiserver
```

#### InfluxDB 恢复
```bash
# 停止服务
docker compose stop influxdb

# 清理数据目录
docker volume rm fonoster_influxdb

# 启动服务
docker compose start influxdb

# 恢复数据
docker cp backup/influxdb_20240101_020000 influxdb:/tmp/
docker exec influxdb influx restore /tmp/influxdb_20240101_020000
```

### 配置备份

#### 备份配置文件
```bash
#!/bin/bash
# backup-config.sh

CONFIG_BACKUP_DIR="/backup/config"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $CONFIG_BACKUP_DIR

# 备份环境变量
cp .env $CONFIG_BACKUP_DIR/env_$DATE

# 备份密钥
cp -r config/keys $CONFIG_BACKUP_DIR/keys_$DATE

# 备份 Docker Compose 文件
cp compose.yaml $CONFIG_BACKUP_DIR/compose_$DATE.yaml

# 备份集成配置
cp config/integrations.json $CONFIG_BACKUP_DIR/integrations_$DATE.json

# 压缩配置备份
tar -czf $CONFIG_BACKUP_DIR/config_backup_$DATE.tar.gz $CONFIG_BACKUP_DIR/*_$DATE*

echo "Configuration backup completed: config_backup_$DATE.tar.gz"
```

## 故障排除

### 常见问题

#### 1. 服务启动失败

**问题**: 容器无法启动
```bash
# 检查日志
docker compose logs <service-name>

# 检查容器状态
docker compose ps

# 检查资源使用
docker stats
```

**解决方案**:
- 检查端口冲突
- 验证环境变量配置
- 确保有足够的系统资源
- 检查 Docker 守护进程状态

#### 2. 数据库连接失败

**问题**: 无法连接到 PostgreSQL
```bash
# 测试数据库连接
docker exec postgres psql -U postgres -c "SELECT version();"

# 检查网络连接
docker network ls
docker network inspect fonoster_default
```

**解决方案**:
- 验证数据库 URL 格式
- 检查数据库服务状态
- 确认网络配置
- 检查防火墙设置

#### 3. SIP 注册失败

**问题**: SIP 客户端无法注册
```bash
# 检查 Routr 日志
docker compose logs routr

# 检查 Asterisk 日志
docker compose logs asterisk

# 测试 SIP 连接
nmap -sU -p 5060 your-server-ip
```

**解决方案**:
- 验证外部 IP 地址配置
- 检查 NAT 和防火墙设置
- 确认 SIP 凭据配置
- 检查网络拓扑

#### 4. 媒体流问题

**问题**: 通话无声音或单向音频
```bash
# 检查 RTPEngine 状态
docker compose logs rtpengine

# 检查端口范围
netstat -ulnp | grep -E "1[0-9]{4}"

# 测试 RTP 端口
nc -u -l 10000
```

**解决方案**:
- 验证 RTP 端口范围配置
- 检查防火墙 UDP 端口
- 确认公网 IP 设置
- 检查网络 QoS 设置

#### 5. 性能问题

**问题**: 系统响应缓慢
```bash
# 检查系统资源
top
htop
iotop

# 检查 Docker 资源使用
docker stats

# 检查数据库性能
docker exec postgres psql -U postgres -c "SELECT * FROM pg_stat_activity;"
```

**解决方案**:
- 增加系统资源
- 优化数据库查询
- 调整容器资源限制
- 启用缓存机制

### 调试工具

#### 网络调试
```bash
# 端口扫描
nmap -p 1-65535 your-server-ip

# 网络连通性测试
telnet your-server-ip 5060
nc -zv your-server-ip 50051

# 抓包分析
tcpdump -i any -w capture.pcap port 5060
wireshark capture.pcap
```

#### 服务调试
```bash
# 进入容器调试
docker exec -it apiserver /bin/bash

# 查看进程状态
docker exec apiserver ps aux

# 检查文件系统
docker exec apiserver df -h
docker exec apiserver ls -la /opt/fonoster/
```

#### 日志分析
```bash
# 实时日志监控
docker compose logs -f --tail=100

# 搜索错误日志
docker compose logs | grep -i error

# 导出日志
docker compose logs > fonoster.log
```

### 性能优化

#### 数据库优化
```sql
-- PostgreSQL 性能调优
ALTER SYSTEM SET shared_buffers = '256MB';
ALTER SYSTEM SET effective_cache_size = '1GB';
ALTER SYSTEM SET maintenance_work_mem = '64MB';
ALTER SYSTEM SET checkpoint_completion_target = 0.9;
ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM SET default_statistics_target = 100;

-- 重启数据库使配置生效
SELECT pg_reload_conf();
```

#### 容器资源限制
```yaml
services:
  apiserver:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
```

#### 缓存配置
```bash
# Redis 缓存
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7-alpine

# 配置应用使用 Redis
APISERVER_CACHE_ENABLED=true
APISERVER_CACHE_URL=redis://redis:6379
```

## 扩展和集成

### 水平扩展

#### API Server 扩展
```yaml
services:
  apiserver:
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

#### 数据库集群
```yaml
# PostgreSQL 主从复制
services:
  postgres-master:
    image: postgres:16.10-alpine3.22
    environment:
      POSTGRES_REPLICATION_MODE: master
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: replicator_password

  postgres-slave:
    image: postgres:16.10-alpine3.22
    environment:
      POSTGRES_REPLICATION_MODE: slave
      POSTGRES_MASTER_HOST: postgres-master
      POSTGRES_REPLICATION_USER: replicator
      POSTGRES_REPLICATION_PASSWORD: replicator_password
```

### 第三方集成

#### 监控集成
```yaml
# Prometheus + Grafana
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
```

#### 日志集成
```yaml
# ELK Stack
services:
  elasticsearch:
    image: elasticsearch:7.14.0
    environment:
      - discovery.type=single-node

  logstash:
    image: logstash:7.14.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf

  kibana:
    image: kibana:7.14.0
    ports:
      - "5601:5601"
```

## 最佳实践

### 安全最佳实践

1. **定期更新**: 保持所有组件为最新版本
2. **最小权限**: 使用最小权限原则配置用户和服务
3. **网络隔离**: 使用防火墙和网络分段
4. **加密传输**: 启用 TLS/SSL 加密
5. **审计日志**: 启用详细的审计日志记录
6. **备份验证**: 定期验证备份的完整性

### 运维最佳实践

1. **监控告警**: 设置全面的监控和告警系统
2. **容量规划**: 根据使用情况进行容量规划
3. **灾难恢复**: 制定和测试灾难恢复计划
4. **文档维护**: 保持部署和运维文档的更新
5. **变更管理**: 建立标准的变更管理流程
6. **性能测试**: 定期进行性能和压力测试

### 开发最佳实践

1. **环境一致性**: 保持开发、测试、生产环境的一致性
2. **配置管理**: 使用配置管理工具管理环境变量
3. **CI/CD 集成**: 集成持续集成和部署流程
4. **代码质量**: 使用代码质量检查工具
5. **测试覆盖**: 保持高测试覆盖率
6. **版本控制**: 使用语义化版本控制

---

本文档提供了 Fonoster 平台的完整部署和配置指南。如需更多帮助，请参考官方文档或联系技术支持团队。