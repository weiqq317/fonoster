# Fonoster 配置指南

本文档详细说明了 Fonoster 平台的配置选项、环境变量和各种部署场景的配置方法。

## 目录

- [概述](#概述)
- [环境变量配置](#环境变量配置)
- [服务配置](#服务配置)
- [网络配置](#网络配置)
- [安全配置](#安全配置)
- [数据库配置](#数据库配置)
- [日志配置](#日志配置)
- [集成配置](#集成配置)
- [性能调优](#性能调优)
- [部署环境配置](#部署环境配置)
- [故障排除](#故障排除)

## 概述

Fonoster 使用环境变量和配置文件来管理不同环境下的配置。主要配置方式包括：

- **环境变量**: 通过 `.env` 文件或系统环境变量
- **配置文件**: JSON/YAML 格式的配置文件
- **Docker Compose**: 容器编排配置
- **Kubernetes**: K8s 资源配置

### 配置优先级

1. 系统环境变量 (最高优先级)
2. `.env` 文件
3. 默认配置值 (最低优先级)

## 环境变量配置

### 核心环境变量

#### 网络配置

```bash
# 必须配置 - 替换为实际的服务器 IP 地址
ROUTR_EXTERNAL_ADDRS=192.168.1.100    # Routr 外部地址
ASTERISK_SIPPROXY_HOST=192.168.1.100  # Asterisk SIP 代理主机
RTPENGINE_PUBLIC_IP=192.168.1.100     # RTP Engine 公网 IP

# 可选网络配置
ROUTR_RTPENGINE_HOST=rtpengine         # RTP Engine 主机名
ASTERISK_SIPPROXY_PORT=5060            # SIP 代理端口
RTPENGINE_PORT_MIN=10000               # RTP 端口范围最小值
RTPENGINE_PORT_MAX=20000               # RTP 端口范围最大值
```

#### API 服务器配置

```bash
# 基础配置
APISERVER_PORT=50051                   # gRPC 服务端口
APISERVER_HOST=0.0.0.0                # 监听地址
APISERVER_LOGS_LEVEL=info             # 日志级别

# 管理员账户
APISERVER_OWNER_EMAIL=admin@fonoster.local
APISERVER_OWNER_PASSWORD=changeme     # 生产环境必须修改

# 认证配置
APISERVER_JWT_SECRET=your-jwt-secret   # JWT 密钥
APISERVER_JWT_EXPIRES_IN=1h           # JWT 过期时间
APISERVER_REFRESH_TOKEN_SECRET=your-refresh-secret
APISERVER_REFRESH_TOKEN_EXPIRES_IN=7d

# 数据库连接
APISERVER_DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fonoster
APISERVER_IDENTITY_DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fnidentity

# 外部服务
APISERVER_ROUTR_API_URL=http://routr:4567
APISERVER_ROUTR_API_USERNAME=admin
APISERVER_ROUTR_API_SECRET=changeme

# 文件存储
APISERVER_STORAGE_DRIVER=local         # local | s3 | gcs
APISERVER_STORAGE_PATH=/tmp/fonoster   # 本地存储路径
```

#### Dashboard 配置

```bash
# 基础配置
DASHBOARD_PORT=3030                    # Web 服务端口
DASHBOARD_HOST=0.0.0.0                # 监听地址

# API 连接
DASHBOARD_API_URL=http://localhost:50051
DASHBOARD_API_TIMEOUT=30000           # API 超时时间 (毫秒)

# 认证配置
DASHBOARD_SESSION_SECRET=your-session-secret
DASHBOARD_SESSION_EXPIRES_IN=24h

# 功能开关
DASHBOARD_ENABLE_SIGNUP=true          # 允许用户注册
DASHBOARD_ENABLE_FORGOT_PASSWORD=true # 允许密码重置
DASHBOARD_ENABLE_ANALYTICS=false      # 启用分析功能
```

#### Autopilot (AI 助手) 配置

```bash
# 基础配置
AUTOPILOT_PORT=3031                   # 服务端口
AUTOPILOT_HOST=0.0.0.0               # 监听地址

# AI 模型配置
AUTOPILOT_OPENAI_API_KEY=sk-...       # OpenAI API 密钥
AUTOPILOT_OPENAI_MODEL=gpt-4         # 使用的模型
AUTOPILOT_OPENAI_MAX_TOKENS=1000     # 最大 token 数
AUTOPILOT_OPENAI_TEMPERATURE=0.7     # 创造性参数

# 语音配置
AUTOPILOT_TTS_PROVIDER=openai        # TTS 提供商
AUTOPILOT_TTS_VOICE=alloy            # 语音类型
AUTOPILOT_STT_PROVIDER=openai        # STT 提供商

# 知识库配置
AUTOPILOT_KNOWLEDGE_BASE_BUCKET=your-s3-bucket
AUTOPILOT_KNOWLEDGE_BASE_REGION=us-east-1
AUTOPILOT_AWS_ACCESS_KEY_ID=your-access-key
AUTOPILOT_AWS_SECRET_ACCESS_KEY=your-secret-key
```

### 基础设施配置

#### PostgreSQL 配置

```bash
# 数据库配置
POSTGRES_USER=postgres                # 数据库用户
POSTGRES_PASSWORD=postgres            # 数据库密码
POSTGRES_DB=fonoster                  # 默认数据库名
POSTGRES_HOST=postgres                # 数据库主机
POSTGRES_PORT=5432                    # 数据库端口

# 连接池配置
POSTGRES_MAX_CONNECTIONS=100          # 最大连接数
POSTGRES_IDLE_TIMEOUT=30000          # 空闲超时 (毫秒)
POSTGRES_CONNECTION_TIMEOUT=60000     # 连接超时 (毫秒)
```

#### InfluxDB 配置

```bash
# InfluxDB 配置
INFLUXDB_INIT_MODE=setup             # 初始化模式
INFLUXDB_INIT_USERNAME=admin         # 管理员用户名
INFLUXDB_INIT_PASSWORD=changeme      # 管理员密码
INFLUXDB_INIT_ORG=fonoster          # 组织名
INFLUXDB_INIT_BUCKET=metrics        # 默认存储桶
INFLUXDB_INIT_ADMIN_TOKEN=your-admin-token

# 数据保留策略
INFLUXDB_RETENTION_POLICY=30d        # 数据保留时间
```

#### NATS 配置

```bash
# NATS 配置
NATS_URL=nats://nats:4222            # NATS 服务器地址
NATS_USER=nats                       # 用户名
NATS_PASSWORD=changeme               # 密码
NATS_CLUSTER_ID=fonoster             # 集群 ID

# 连接配置
NATS_MAX_RECONNECT_ATTEMPTS=10       # 最大重连次数
NATS_RECONNECT_TIME_WAIT=2000        # 重连等待时间 (毫秒)
```

### Routr 配置

```bash
# Routr 基础配置
ROUTR_API_PORT=4567                  # API 端口
ROUTR_SIP_PORT=5060                  # SIP 端口
ROUTR_API_USERNAME=admin             # API 用户名
ROUTR_API_SECRET=changeme            # API 密码

# 外部地址配置
ROUTR_EXTERNAL_ADDRS=192.168.1.100   # 外部 IP 地址
ROUTR_LOCALNETS=192.168.1.0/24       # 本地网络

# 数据库配置
ROUTR_DS_PROVIDER=redis_data_provider # 数据源提供商
ROUTR_DS_PARAMETERS=redis://redis:6379/0

# 日志配置
ROUTR_LOGS_LEVEL=info                # 日志级别
ROUTR_LOGS_TRANSPORT=console         # 日志输出方式

# NATS 发布者配置
ROUTR_NATS_PUBLISHER_ENABLED=true    # 启用 NATS 发布
ROUTR_NATS_PUBLISHER_URL=nats://nats:4222
```

### Asterisk 配置

```bash
# Asterisk 基础配置
ASTERISK_ARI_PROXY_HOST=asterisk     # ARI 代理主机
ASTERISK_ARI_PROXY_PORT=8088         # ARI 代理端口
ASTERISK_ARI_SECRET=changeme         # ARI 密钥
ASTERISK_ARI_USERNAME=admin          # ARI 用户名

# 音频配置
ASTERISK_CODECS=ulaw,alaw,g722       # 支持的编解码器
ASTERISK_DTMF_MODE=rfc2833           # DTMF 模式

# RTP 配置
ASTERISK_RTP_PORT_START=10000        # RTP 端口起始
ASTERISK_RTP_PORT_END=20000          # RTP 端口结束

# SIP 代理配置
ASTERISK_SIPPROXY_HOST=192.168.1.100 # SIP 代理主机
ASTERISK_SIPPROXY_PORT=5060          # SIP 代理端口
ASTERISK_SIPPROXY_SECRET=changeme    # SIP 代理密钥
ASTERISK_SIPPROXY_USERNAME=admin     # SIP 代理用户名
```

### 开发环境配置

```bash
# 开发模式
NODE_ENV=development                 # 环境模式
LOGS_LEVEL=verbose                   # 详细日志

# 热重载配置
NODEMON_WATCH=mods/**               # 监听文件变化
NODEMON_EXT=ts,js,json              # 监听文件扩展名
NODEMON_IGNORE=node_modules/**      # 忽略目录

# 调试配置
DEBUG=fonoster:*                    # 调试命名空间
DEBUG_COLORS=true                   # 启用颜色输出
```

## 服务配置

### API 服务器高级配置

#### Prisma 配置

```bash
# Prisma 配置
DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fonoster
SHADOW_DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fonoster_shadow

# 连接池配置
PRISMA_CONNECTION_LIMIT=10          # 连接池大小
PRISMA_POOL_TIMEOUT=20              # 连接超时 (秒)
```

#### gRPC 配置

```typescript
// mods/apiserver/src/config/grpc.ts
export const grpcConfig = {
  port: process.env.APISERVER_PORT || 50051,
  host: process.env.APISERVER_HOST || '0.0.0.0',
  options: {
    'grpc.keepalive_time_ms': 30000,
    'grpc.keepalive_timeout_ms': 5000,
    'grpc.keepalive_permit_without_calls': true,
    'grpc.http2.max_pings_without_data': 0,
    'grpc.http2.min_time_between_pings_ms': 10000,
    'grpc.http2.min_ping_interval_without_data_ms': 300000
  }
};
```

### Voice 服务配置

```bash
# Voice 服务配置
VOICE_PORT=3032                     # 服务端口
VOICE_HOST=0.0.0.0                 # 监听地址

# Asterisk 连接
VOICE_ASTERISK_ARI_URL=http://asterisk:8088
VOICE_ASTERISK_ARI_USERNAME=admin
VOICE_ASTERISK_ARI_PASSWORD=changeme

# 音频处理
VOICE_AUDIO_FORMAT=wav              # 音频格式
VOICE_AUDIO_SAMPLE_RATE=8000       # 采样率
VOICE_AUDIO_CHANNELS=1             # 声道数

# 录音配置
VOICE_RECORDING_ENABLED=true       # 启用录音
VOICE_RECORDING_PATH=/tmp/recordings
VOICE_RECORDING_MAX_DURATION=3600  # 最大录音时长 (秒)
```

### Identity 服务配置

```bash
# Identity 服务配置
IDENTITY_PORT=50052                # 服务端口
IDENTITY_HOST=0.0.0.0             # 监听地址

# JWT 配置
IDENTITY_JWT_ALGORITHM=RS256       # JWT 算法
IDENTITY_JWT_ISSUER=fonoster       # JWT 签发者
IDENTITY_JWT_AUDIENCE=fonoster-api # JWT 受众

# 密码策略
IDENTITY_PASSWORD_MIN_LENGTH=8     # 最小密码长度
IDENTITY_PASSWORD_REQUIRE_UPPERCASE=true
IDENTITY_PASSWORD_REQUIRE_LOWERCASE=true
IDENTITY_PASSWORD_REQUIRE_NUMBERS=true
IDENTITY_PASSWORD_REQUIRE_SYMBOLS=false

# 账户锁定策略
IDENTITY_MAX_LOGIN_ATTEMPTS=5      # 最大登录尝试次数
IDENTITY_LOCKOUT_DURATION=900      # 锁定时长 (秒)
```

## 网络配置

### Docker 网络配置

```yaml
# compose.yaml
networks:
  fonoster:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### Envoy 代理配置

```yaml
# config/envoy.yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 51051
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: grpc_json
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  cluster: apiserver
              cors:
                allow_origin_string_match:
                - prefix: "*"
                allow_methods: GET, PUT, DELETE, POST, OPTIONS
                allow_headers: keep-alive,user-agent,cache-control,content-type,content-transfer-encoding,custom-header-1,x-accept-content-transfer-encoding,x-accept-response-streaming,x-user-agent,x-grpc-web,grpc-timeout
                max_age: "1728000"
                expose_headers: custom-header-1,grpc-status,grpc-message
          http_filters:
          - name: envoy.filters.http.grpc_web
          - name: envoy.filters.http.cors
          - name: envoy.filters.http.router

  clusters:
  - name: apiserver
    connect_timeout: 0.25s
    type: logical_dns
    http2_protocol_options: {}
    lb_policy: round_robin
    load_assignment:
      cluster_name: apiserver
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: apiserver
                port_value: 50051
```

### 防火墙配置

```bash
# 开放必要端口
# Web 服务
3030/tcp    # Dashboard
3031/tcp    # Autopilot
3032/tcp    # Voice

# API 服务
50051/tcp   # gRPC API
51051/tcp   # gRPC-Web (Envoy)

# SIP 服务
5060/tcp    # SIP
5060/udp    # SIP

# RTP 服务
10000-20000/udp  # RTP 媒体流

# 管理服务
8282/tcp    # Adminer
8025/tcp    # MailHog
```

## 安全配置

### SSL/TLS 配置

#### 生成自签名证书

```bash
# 生成 RSA 密钥对
mkdir -p .keys
openssl genrsa -out .keys/private.pem 2048
openssl rsa -in .keys/private.pem -pubout -out .keys/public.pem

# 生成 SSL 证书
openssl req -new -x509 -key .keys/private.pem -out .keys/cert.pem -days 365
```

#### 配置 HTTPS

```bash
# 启用 HTTPS
DASHBOARD_HTTPS_ENABLED=true
DASHBOARD_SSL_CERT_PATH=.keys/cert.pem
DASHBOARD_SSL_KEY_PATH=.keys/private.pem

# 强制 HTTPS 重定向
DASHBOARD_FORCE_HTTPS=true
```

### 认证和授权

```bash
# API 密钥配置
APISERVER_API_KEY_ENABLED=true
APISERVER_API_KEY_HEADER=x-api-key

# OAuth 配置
APISERVER_OAUTH_ENABLED=false
APISERVER_OAUTH_GOOGLE_CLIENT_ID=your-google-client-id
APISERVER_OAUTH_GOOGLE_CLIENT_SECRET=your-google-client-secret

# RBAC 配置
APISERVER_RBAC_ENABLED=true
APISERVER_RBAC_DEFAULT_ROLE=user
```

### 数据加密

```bash
# 数据库加密
POSTGRES_SSL_MODE=require
POSTGRES_SSL_CERT_PATH=.keys/postgres-cert.pem
POSTGRES_SSL_KEY_PATH=.keys/postgres-key.pem

# 传输加密
APISERVER_TLS_ENABLED=true
APISERVER_TLS_CERT_PATH=.keys/server-cert.pem
APISERVER_TLS_KEY_PATH=.keys/server-key.pem
```

## 数据库配置

### PostgreSQL 优化

```bash
# 连接配置
POSTGRES_MAX_CONNECTIONS=200
POSTGRES_SHARED_BUFFERS=256MB
POSTGRES_EFFECTIVE_CACHE_SIZE=1GB
POSTGRES_MAINTENANCE_WORK_MEM=64MB
POSTGRES_CHECKPOINT_COMPLETION_TARGET=0.9
POSTGRES_WAL_BUFFERS=16MB
POSTGRES_DEFAULT_STATISTICS_TARGET=100

# 日志配置
POSTGRES_LOG_STATEMENT=all
POSTGRES_LOG_DURATION=on
POSTGRES_LOG_MIN_DURATION_STATEMENT=1000
```

### 数据库备份配置

```bash
# 备份配置
BACKUP_ENABLED=true
BACKUP_SCHEDULE="0 2 * * *"          # 每天凌晨 2 点
BACKUP_RETENTION_DAYS=30             # 保留 30 天
BACKUP_S3_BUCKET=fonoster-backups    # S3 存储桶
BACKUP_S3_REGION=us-east-1           # S3 区域
```

### Redis 配置 (可选)

```bash
# Redis 配置
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=changeme
REDIS_DB=0

# 缓存配置
CACHE_TTL=3600                       # 缓存过期时间 (秒)
CACHE_MAX_SIZE=1000                  # 最大缓存条目数
```

## 日志配置

### 日志级别

```bash
# 全局日志级别
LOGS_LEVEL=info                      # error | warn | info | verbose | debug

# 服务特定日志级别
APISERVER_LOGS_LEVEL=info
DASHBOARD_LOGS_LEVEL=warn
AUTOPILOT_LOGS_LEVEL=debug
VOICE_LOGS_LEVEL=info
```

### 日志格式

```bash
# 日志格式
LOGS_FORMAT=json                     # json | text
LOGS_TIMESTAMP=true                  # 包含时间戳
LOGS_COLORIZE=false                  # 生产环境关闭颜色

# 日志输出
LOGS_CONSOLE=true                    # 控制台输出
LOGS_FILE=false                      # 文件输出
LOGS_FILE_PATH=/var/log/fonoster     # 日志文件路径
```

### 结构化日志

```typescript
// 日志配置示例
import { createLogger, format, transports } from 'winston';

const logger = createLogger({
  level: process.env.LOGS_LEVEL || 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json()
  ),
  defaultMeta: { service: 'apiserver' },
  transports: [
    new transports.Console({
      format: format.combine(
        format.colorize(),
        format.simple()
      )
    }),
    new transports.File({ 
      filename: 'error.log', 
      level: 'error' 
    }),
    new transports.File({ 
      filename: 'combined.log' 
    })
  ]
});
```

### 日志聚合

```bash
# ELK Stack 配置
ELASTICSEARCH_URL=http://elasticsearch:9200
LOGSTASH_HOST=logstash
LOGSTASH_PORT=5000
KIBANA_URL=http://kibana:5601

# Fluentd 配置
FLUENTD_HOST=fluentd
FLUENTD_PORT=24224
FLUENTD_TAG=fonoster
```

## 集成配置

### 第三方服务集成

#### AWS 服务

```bash
# AWS 基础配置
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key

# S3 配置
AWS_S3_BUCKET=fonoster-storage
AWS_S3_REGION=us-east-1
AWS_S3_ENDPOINT=https://s3.amazonaws.com

# SES 配置 (邮件服务)
AWS_SES_REGION=us-east-1
AWS_SES_FROM_EMAIL=noreply@yourdomain.com
AWS_SES_REPLY_TO_EMAIL=support@yourdomain.com
```

#### OpenAI 集成

```bash
# OpenAI API 配置
OPENAI_API_KEY=sk-your-api-key
OPENAI_ORGANIZATION=org-your-org-id
OPENAI_MODEL=gpt-4
OPENAI_MAX_TOKENS=2000
OPENAI_TEMPERATURE=0.7

# 语音服务
OPENAI_TTS_MODEL=tts-1
OPENAI_TTS_VOICE=alloy
OPENAI_STT_MODEL=whisper-1
```

#### Twilio 集成

```bash
# Twilio 配置
TWILIO_ACCOUNT_SID=your-account-sid
TWILIO_AUTH_TOKEN=your-auth-token
TWILIO_PHONE_NUMBER=+1234567890

# Webhook 配置
TWILIO_WEBHOOK_URL=https://yourdomain.com/webhook
TWILIO_WEBHOOK_SECRET=your-webhook-secret
```

### Webhook 配置

```bash
# Webhook 基础配置
WEBHOOK_ENABLED=true
WEBHOOK_URL=https://yourdomain.com/webhook
WEBHOOK_SECRET=your-webhook-secret
WEBHOOK_TIMEOUT=30000               # 超时时间 (毫秒)

# 重试配置
WEBHOOK_RETRY_ATTEMPTS=3            # 重试次数
WEBHOOK_RETRY_DELAY=1000           # 重试延迟 (毫秒)
WEBHOOK_RETRY_BACKOFF=2            # 退避倍数

# 事件过滤
WEBHOOK_EVENTS=call.started,call.ended,call.answered
```

## 性能调优

### 应用性能配置

```bash
# Node.js 性能配置
NODE_OPTIONS="--max-old-space-size=4096"  # 最大堆内存
UV_THREADPOOL_SIZE=16                     # 线程池大小

# 集群配置
CLUSTER_ENABLED=true                      # 启用集群模式
CLUSTER_WORKERS=4                         # 工作进程数

# 缓存配置
CACHE_ENABLED=true                        # 启用缓存
CACHE_TTL=3600                           # 缓存过期时间
CACHE_MAX_SIZE=10000                     # 最大缓存条目
```

### 数据库性能

```bash
# 连接池配置
DB_POOL_MIN=5                            # 最小连接数
DB_POOL_MAX=20                           # 最大连接数
DB_POOL_IDLE_TIMEOUT=30000               # 空闲超时
DB_POOL_ACQUIRE_TIMEOUT=60000            # 获取连接超时

# 查询优化
DB_QUERY_TIMEOUT=30000                   # 查询超时
DB_STATEMENT_TIMEOUT=60000               # 语句超时
```

### 媒体处理性能

```bash
# RTP 配置
RTP_BUFFER_SIZE=1024                     # RTP 缓冲区大小
RTP_JITTER_BUFFER=50                     # 抖动缓冲区
RTP_PACKET_SIZE=160                      # RTP 包大小

# 音频处理
AUDIO_PROCESSING_THREADS=4               # 音频处理线程数
AUDIO_BUFFER_SIZE=4096                   # 音频缓冲区大小
```

## 部署环境配置

### 开发环境

```bash
# .env.development
NODE_ENV=development
LOGS_LEVEL=verbose
DEBUG=fonoster:*

# 开发服务配置
APISERVER_HOST=0.0.0.0
DASHBOARD_HOST=0.0.0.0
AUTOPILOT_HOST=0.0.0.0

# 热重载
NODEMON_ENABLED=true
WEBPACK_DEV_SERVER=true
```

### 测试环境

```bash
# .env.test
NODE_ENV=test
LOGS_LEVEL=error

# 测试数据库
POSTGRES_DB=fonoster_test
APISERVER_DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fonoster_test

# 禁用外部服务
WEBHOOK_ENABLED=false
EMAIL_ENABLED=false
SMS_ENABLED=false
```

### 预生产环境

```bash
# .env.staging
NODE_ENV=staging
LOGS_LEVEL=info

# 使用生产级配置但保留调试功能
DEBUG_ENABLED=true
PROFILING_ENABLED=true

# 限制资源使用
RATE_LIMIT_ENABLED=true
RATE_LIMIT_MAX=1000
```

### 生产环境

```bash
# .env.production
NODE_ENV=production
LOGS_LEVEL=warn

# 安全配置
HTTPS_ENABLED=true
FORCE_HTTPS=true
SECURE_COOKIES=true
CSRF_PROTECTION=true

# 性能配置
CLUSTER_ENABLED=true
CACHE_ENABLED=true
COMPRESSION_ENABLED=true

# 监控配置
MONITORING_ENABLED=true
METRICS_ENABLED=true
HEALTH_CHECK_ENABLED=true
```

### Kubernetes 配置

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fonoster-config
data:
  NODE_ENV: "production"
  LOGS_LEVEL: "info"
  APISERVER_PORT: "50051"
  DASHBOARD_PORT: "3030"
  
---
apiVersion: v1
kind: Secret
metadata:
  name: fonoster-secrets
type: Opaque
stringData:
  POSTGRES_PASSWORD: "your-secure-password"
  JWT_SECRET: "your-jwt-secret"
  OPENAI_API_KEY: "your-openai-key"
```

## 故障排除

### 常见配置问题

#### 1. 网络连接问题

```bash
# 检查网络配置
docker network ls
docker network inspect fonoster_default

# 测试服务连接
docker compose exec apiserver ping postgres
docker compose exec apiserver ping routr

# 检查端口占用
netstat -tulpn | grep :50051
lsof -i :3030
```

#### 2. 数据库连接问题

```bash
# 检查数据库状态
docker compose exec postgres pg_isready

# 测试数据库连接
docker compose exec postgres psql -U postgres -d fonoster -c "SELECT 1;"

# 检查数据库日志
docker compose logs postgres
```

#### 3. 环境变量问题

```bash
# 检查环境变量
docker compose exec apiserver env | grep APISERVER
docker compose exec apiserver env | grep POSTGRES

# 验证配置文件
docker compose config
```

#### 4. 服务启动问题

```bash
# 检查服务状态
docker compose ps
docker compose logs -f apiserver

# 重启特定服务
docker compose restart apiserver
docker compose up -d --force-recreate apiserver
```

### 配置验证

#### 自动化配置检查

```bash
#!/bin/bash
# scripts/check-config.sh

echo "检查配置文件..."

# 检查必需的环境变量
required_vars=(
  "ROUTR_EXTERNAL_ADDRS"
  "ASTERISK_SIPPROXY_HOST" 
  "RTPENGINE_PUBLIC_IP"
  "APISERVER_OWNER_EMAIL"
  "POSTGRES_PASSWORD"
)

for var in "${required_vars[@]}"; do
  if [ -z "${!var}" ]; then
    echo "错误: 缺少必需的环境变量 $var"
    exit 1
  fi
done

echo "配置检查通过!"
```

#### 健康检查端点

```typescript
// 健康检查 API
app.get('/health', (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    services: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      nats: await checkNats()
    }
  };
  
  res.json(health);
});
```

### 性能监控

```bash
# 启用性能监控
MONITORING_ENABLED=true
METRICS_ENDPOINT=/metrics
HEALTH_CHECK_ENDPOINT=/health

# Prometheus 配置
PROMETHEUS_PORT=9090
PROMETHEUS_SCRAPE_INTERVAL=15s

# Grafana 配置  
GRAFANA_PORT=3000
GRAFANA_ADMIN_PASSWORD=changeme
```

---

## 总结

本配置指南涵盖了 Fonoster 平台的所有主要配置选项。在部署时，请根据你的具体环境和需求调整相应的配置参数。

### 配置最佳实践

1. **安全第一**: 在生产环境中更改所有默认密码和密钥
2. **环境隔离**: 为不同环境使用不同的配置文件
3. **配置验证**: 部署前验证所有配置参数
4. **监控配置**: 启用适当的日志和监控
5. **备份配置**: 定期备份配置文件和数据库
6. **文档更新**: 保持配置文档与实际部署同步

如需更多帮助，请参考 [部署指南](deployment.md) 或在 [GitHub Issues](https://github.com/fonoster/fonoster/issues) 中提问。