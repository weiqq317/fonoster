# Fonoster 故障排除和问题解决指南

本文档提供了 Fonoster 平台常见问题的诊断方法和解决方案，帮助开发者和运维人员快速定位和解决问题。

## 目录

- [快速诊断](#快速诊断)
- [服务启动问题](#服务启动问题)
- [网络连接问题](#网络连接问题)
- [数据库问题](#数据库问题)
- [认证和授权问题](#认证和授权问题)
- [语音通话问题](#语音通话问题)
- [性能问题](#性能问题)
- [日志分析](#日志分析)
- [监控和诊断工具](#监控和诊断工具)
- [常见错误代码](#常见错误代码)
- [紧急恢复程序](#紧急恢复程序)

## 快速诊断

### 系统健康检查

```bash
#!/bin/bash
# 快速健康检查脚本

echo "=== Fonoster 系统健康检查 ==="

# 1. 检查 Docker 服务状态
echo "1. 检查 Docker 服务..."
docker compose ps

# 2. 检查关键端口
echo "2. 检查关键端口..."
ports=(3030 50051 51051 5060 8088)
for port in "${ports[@]}"; do
  if lsof -i :$port > /dev/null 2>&1; then
    echo "✓ 端口 $port 正在使用"
  else
    echo "✗ 端口 $port 未使用"
  fi
done

# 3. 检查数据库连接
echo "3. 检查数据库连接..."
if docker compose exec -T postgres pg_isready > /dev/null 2>&1; then
  echo "✓ PostgreSQL 连接正常"
else
  echo "✗ PostgreSQL 连接失败"
fi

# 4. 检查 API 服务
echo "4. 检查 API 服务..."
if curl -s http://localhost:51051/health > /dev/null 2>&1; then
  echo "✓ API 服务正常"
else
  echo "✗ API 服务异常"
fi

# 5. 检查磁盘空间
echo "5. 检查磁盘空间..."
df -h | grep -E "(/$|/var|/tmp)"

echo "=== 健康检查完成 ==="
```

### 快速修复命令

```bash
# 重启所有服务
docker compose restart

# 重建并启动服务
docker compose down && docker compose up -d --build

# 清理并重新开始
docker compose down -v
docker system prune -f
docker compose up -d
```

## 服务启动问题

### API 服务器启动失败

#### 问题症状
- API 服务器无法启动
- gRPC 连接被拒绝
- 端口 50051 无响应

#### 诊断步骤

```bash
# 1. 检查服务状态
docker compose ps apiserver

# 2. 查看详细日志
docker compose logs -f apiserver

# 3. 检查端口占用
lsof -i :50051
netstat -tulpn | grep 50051

# 4. 检查环境变量
docker compose exec apiserver env | grep APISERVER

# 5. 检查数据库连接
docker compose exec apiserver npm run db:check
```

#### 常见原因和解决方案

**1. 端口被占用**
```bash
# 查找占用进程
lsof -i :50051
# 终止占用进程
kill -9 <PID>
# 或修改端口配置
export APISERVER_PORT=50052
```

**2. 数据库连接失败**
```bash
# 检查数据库状态
docker compose ps postgres
docker compose logs postgres

# 重启数据库
docker compose restart postgres

# 等待数据库就绪
docker compose exec postgres pg_isready -U postgres
```

**3. 环境变量缺失**
```bash
# 检查 .env 文件
cat .env | grep -E "(APISERVER|POSTGRES|JWT)"

# 复制环境变量模板
cp .env.example .env

# 生成必要的密钥
npm run generate:keypair
```

**4. Prisma 迁移问题**
```bash
# 重置数据库
cd mods/apiserver
npx prisma migrate reset --force

# 手动运行迁移
npx prisma migrate deploy

# 生成 Prisma 客户端
npx prisma generate
```

### Dashboard 启动失败

#### 问题症状
- Web 界面无法访问
- 端口 3030 无响应
- 白屏或加载错误

#### 诊断步骤

```bash
# 1. 检查 Dashboard 服务
docker compose ps dashboard

# 2. 查看启动日志
docker compose logs -f dashboard

# 3. 检查构建状态
docker compose exec dashboard npm run build

# 4. 检查 API 连接
curl -v http://localhost:51051/health
```

#### 解决方案

**1. 构建失败**
```bash
# 清理构建缓存
docker compose exec dashboard rm -rf node_modules/.cache
docker compose exec dashboard npm run clean

# 重新构建
docker compose restart dashboard
```

**2. API 连接问题**
```bash
# 检查 Envoy 代理状态
docker compose ps envoy
docker compose logs envoy

# 重启 Envoy
docker compose restart envoy
```

### Autopilot 服务问题

#### 问题症状
- AI 助手无响应
- OpenAI API 调用失败
- 语音识别/合成错误

#### 诊断步骤

```bash
# 1. 检查服务状态
docker compose ps autopilot

# 2. 检查 OpenAI 配置
docker compose exec autopilot env | grep OPENAI

# 3. 测试 API 连接
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/models
```

#### 解决方案

**1. API 密钥问题**
```bash
# 验证 API 密钥
export OPENAI_API_KEY="sk-your-actual-key"

# 更新配置
docker compose restart autopilot
```

**2. 配额限制**
```bash
# 检查 API 使用情况
curl -H "Authorization: Bearer $OPENAI_API_KEY" \
  https://api.openai.com/v1/usage
```

## 网络连接问题

### SIP 连接问题

#### 问题症状
- SIP 注册失败
- 无法建立通话
- 音频单向或无音频

#### 诊断步骤

```bash
# 1. 检查 Routr 状态
docker compose ps routr
docker compose logs routr

# 2. 检查 SIP 端口
netstat -ulpn | grep 5060
telnet <ROUTR_EXTERNAL_ADDRS> 5060

# 3. 检查 Asterisk 连接
docker compose exec asterisk asterisk -rx "sip show peers"
docker compose exec asterisk asterisk -rx "core show channels"

# 4. 检查网络配置
docker network inspect fonoster_default
```

#### 解决方案

**1. IP 地址配置错误**
```bash
# 获取本机 IP
ip route get 8.8.8.8 | awk '{print $7; exit}'

# 更新环境变量
export ROUTR_EXTERNAL_ADDRS=<your-ip>
export ASTERISK_SIPPROXY_HOST=<your-ip>
export RTPENGINE_PUBLIC_IP=<your-ip>

# 重启相关服务
docker compose restart routr asterisk rtpengine
```

**2. 防火墙问题**
```bash
# 检查防火墙状态 (Linux)
sudo ufw status
sudo iptables -L

# 开放必要端口
sudo ufw allow 5060/tcp
sudo ufw allow 5060/udp
sudo ufw allow 10000:20000/udp

# macOS 防火墙
sudo pfctl -sr | grep 5060
```

**3. NAT 穿透问题**
```bash
# 检查 STUN 服务器配置
docker compose exec routr cat /opt/routr/config/config.yml | grep stun

# 配置 STUN 服务器
# 在 Routr 配置中添加:
# stun:
#   enabled: true
#   host: stun.l.google.com
#   port: 19302
```

### RTP 媒体流问题

#### 问题症状
- 通话建立但无音频
- 音频质量差或断续
- 单向音频

#### 诊断步骤

```bash
# 1. 检查 RTPEngine 状态
docker compose ps rtpengine
docker compose logs rtpengine

# 2. 检查 RTP 端口范围
netstat -ulpn | grep -E "(1[0-9]{4}|20000)"

# 3. 监控 RTP 流量
sudo tcpdump -i any -n port 10000-20000

# 4. 检查音频编解码器
docker compose exec asterisk asterisk -rx "core show codecs"
```

#### 解决方案

**1. RTP 端口被阻塞**
```bash
# 检查端口范围配置
echo "RTP 端口范围: $RTPENGINE_PORT_MIN-$RTPENGINE_PORT_MAX"

# 开放 RTP 端口范围
sudo ufw allow 10000:20000/udp

# 检查 Docker 网络
docker network inspect fonoster_default
```

**2. 编解码器不匹配**
```bash
# 检查支持的编解码器
docker compose exec asterisk asterisk -rx "core show codecs"

# 配置编解码器优先级
# 在 Asterisk 配置中设置:
# allow=ulaw,alaw,g722
```

### gRPC 连接问题

#### 问题症状
- gRPC 调用超时
- 连接被拒绝
- TLS 握手失败

#### 诊断步骤

```bash
# 1. 测试 gRPC 连接
grpcurl -plaintext localhost:50051 list

# 2. 检查 TLS 配置
openssl s_client -connect localhost:50051 -servername localhost

# 3. 测试 gRPC-Web
curl -v http://localhost:51051/health

# 4. 检查 Envoy 配置
docker compose exec envoy cat /etc/envoy/envoy.yaml
```

#### 解决方案

**1. 证书问题**
```bash
# 重新生成证书
openssl genrsa -out .keys/private.pem 2048
openssl rsa -in .keys/private.pem -pubout -out .keys/public.pem

# 重启服务
docker compose restart apiserver envoy
```

**2. Envoy 配置问题**
```bash
# 验证 Envoy 配置
docker compose exec envoy envoy --mode validate --config-path /etc/envoy/envoy.yaml

# 重新加载配置
docker compose restart envoy
```

## 数据库问题

### PostgreSQL 连接失败

#### 问题症状
- 数据库连接超时
- 认证失败
- 连接池耗尽

#### 诊断步骤

```bash
# 1. 检查数据库状态
docker compose ps postgres
docker compose exec postgres pg_isready -U postgres

# 2. 检查连接配置
docker compose exec postgres psql -U postgres -c "\conninfo"

# 3. 查看活动连接
docker compose exec postgres psql -U postgres -c "SELECT * FROM pg_stat_activity;"

# 4. 检查数据库日志
docker compose logs postgres | tail -50
```

#### 解决方案

**1. 连接池配置**
```bash
# 增加最大连接数
export POSTGRES_MAX_CONNECTIONS=200

# 调整连接池参数
export DB_POOL_MAX=20
export DB_POOL_MIN=5

# 重启数据库
docker compose restart postgres
```

**2. 权限问题**
```bash
# 重置数据库权限
docker compose exec postgres psql -U postgres -c "
  GRANT ALL PRIVILEGES ON DATABASE fonoster TO postgres;
  GRANT ALL PRIVILEGES ON DATABASE fnidentity TO postgres;
"
```

**3. 磁盘空间不足**
```bash
# 检查磁盘空间
df -h
docker system df

# 清理 Docker 空间
docker system prune -f
docker volume prune -f
```

### 数据库迁移问题

#### 问题症状
- Prisma 迁移失败
- 表结构不匹配
- 数据丢失

#### 诊断步骤

```bash
# 1. 检查迁移状态
cd mods/apiserver
npx prisma migrate status

# 2. 查看迁移历史
npx prisma migrate diff --from-empty --to-schema-datamodel schema.prisma

# 3. 检查数据库模式
npx prisma db pull
```

#### 解决方案

**1. 迁移冲突**
```bash
# 重置迁移
npx prisma migrate reset --force

# 手动应用迁移
npx prisma migrate deploy

# 生成客户端
npx prisma generate
```

**2. 模式不同步**
```bash
# 推送模式到数据库
npx prisma db push

# 从数据库拉取模式
npx prisma db pull
```

## 认证和授权问题

### JWT 令牌问题

#### 问题症状
- 令牌验证失败
- 令牌过期
- 无效签名

#### 诊断步骤

```bash
# 1. 检查 JWT 配置
docker compose exec apiserver env | grep JWT

# 2. 验证密钥文件
ls -la .keys/
cat .keys/public.pem

# 3. 测试令牌生成
curl -X POST http://localhost:51051/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@fonoster.local","password":"changeme"}'
```

#### 解决方案

**1. 密钥不匹配**
```bash
# 重新生成密钥对
npm run generate:keypair

# 重启 API 服务器
docker compose restart apiserver
```

**2. 时钟同步问题**
```bash
# 检查系统时间
date
docker compose exec apiserver date

# 同步时间 (Linux)
sudo ntpdate -s time.nist.gov
```

### API 密钥认证问题

#### 问题症状
- API 密钥无效
- 权限不足
- 密钥过期

#### 诊断步骤

```bash
# 1. 检查 API 密钥配置
docker compose exec apiserver env | grep API_KEY

# 2. 验证密钥格式
echo $API_KEY | base64 -d

# 3. 测试 API 调用
curl -H "x-api-key: $API_KEY" http://localhost:51051/users
```

#### 解决方案

**1. 重新生成 API 密钥**
```bash
# 生成新的 API 密钥
openssl rand -base64 32

# 更新配置
export APISERVER_API_KEY="your-new-key"
docker compose restart apiserver
```

## 语音通话问题

### 通话建立失败

#### 问题症状
- 呼叫无响应
- SIP 注册失败
- 通话立即断开

#### 诊断步骤

```bash
# 1. 检查 SIP 注册状态
docker compose exec asterisk asterisk -rx "sip show registry"
docker compose exec asterisk asterisk -rx "sip show peers"

# 2. 监控 SIP 消息
docker compose exec routr tail -f /var/log/routr/routr.log

# 3. 检查拨号计划
docker compose exec asterisk asterisk -rx "dialplan show"

# 4. 查看通话记录
docker compose exec asterisk asterisk -rx "core show channels"
```

#### 解决方案

**1. SIP 配置问题**
```bash
# 检查 SIP 配置
docker compose exec asterisk cat /etc/asterisk/sip.conf

# 重新加载 SIP 配置
docker compose exec asterisk asterisk -rx "sip reload"
```

**2. 拨号计划错误**
```bash
# 检查拨号计划语法
docker compose exec asterisk asterisk -rx "dialplan reload"

# 测试拨号计划
docker compose exec asterisk asterisk -rx "dialplan show @default"
```

### 音频质量问题

#### 问题症状
- 音频断续
- 回音或噪音
- 音量过低或过高

#### 诊断步骤

```bash
# 1. 检查音频编解码器
docker compose exec asterisk asterisk -rx "core show codecs"

# 2. 监控 RTP 统计
docker compose exec asterisk asterisk -rx "rtp show stats"

# 3. 检查网络延迟
ping -c 10 $ROUTR_EXTERNAL_ADDRS

# 4. 监控系统资源
docker stats
```

#### 解决方案

**1. 编解码器优化**
```bash
# 配置高质量编解码器
# 在 Asterisk 配置中设置:
# allow=g722,ulaw,alaw
# disallow=all
```

**2. 网络优化**
```bash
# 增加 RTP 缓冲区
# 在 RTPEngine 配置中设置:
# --buffer-size=1024
# --jitter-buffer=50
```

### 录音功能问题

#### 问题症状
- 录音文件为空
- 录音格式错误
- 录音权限问题

#### 诊断步骤

```bash
# 1. 检查录音目录
docker compose exec asterisk ls -la /var/spool/asterisk/monitor/

# 2. 检查录音配置
docker compose exec asterisk cat /etc/asterisk/features.conf

# 3. 测试录音功能
docker compose exec asterisk asterisk -rx "core show applications like record"
```

#### 解决方案

**1. 权限问题**
```bash
# 修复录音目录权限
docker compose exec asterisk chown -R asterisk:asterisk /var/spool/asterisk/monitor/
docker compose exec asterisk chmod 755 /var/spool/asterisk/monitor/
```

**2. 存储空间不足**
```bash
# 检查存储空间
docker compose exec asterisk df -h /var/spool/asterisk/

# 清理旧录音文件
docker compose exec asterisk find /var/spool/asterisk/monitor/ -name "*.wav" -mtime +30 -delete
```

## 性能问题

### 高 CPU 使用率

#### 问题症状
- 系统响应缓慢
- CPU 使用率持续高于 80%
- 服务超时

#### 诊断步骤

```bash
# 1. 检查系统负载
top
htop
docker stats

# 2. 分析进程 CPU 使用
docker compose exec apiserver top
docker compose exec asterisk top

# 3. 检查 Node.js 性能
docker compose exec apiserver node --prof app.js
```

#### 解决方案

**1. 扩展资源**
```bash
# 增加 Node.js 内存限制
export NODE_OPTIONS="--max-old-space-size=4096"

# 启用集群模式
export CLUSTER_ENABLED=true
export CLUSTER_WORKERS=4

# 重启服务
docker compose restart apiserver
```

**2. 优化配置**
```bash
# 启用缓存
export CACHE_ENABLED=true
export CACHE_TTL=3600

# 调整连接池
export DB_POOL_MAX=10
export DB_POOL_MIN=2
```

### 内存泄漏

#### 问题症状
- 内存使用持续增长
- 服务崩溃 (OOM)
- 垃圾回收频繁

#### 诊断步骤

```bash
# 1. 监控内存使用
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# 2. 生成堆转储
docker compose exec apiserver kill -USR2 <pid>

# 3. 分析内存使用
docker compose exec apiserver node --inspect app.js
```

#### 解决方案

**1. 内存限制**
```bash
# 设置容器内存限制
# 在 docker-compose.yml 中添加:
# deploy:
#   resources:
#     limits:
#       memory: 2G
```

**2. 垃圾回收优化**
```bash
# 启用垃圾回收日志
export NODE_OPTIONS="--trace-gc --trace-gc-verbose"

# 调整垃圾回收参数
export NODE_OPTIONS="--max-old-space-size=2048 --gc-interval=100"
```

### 数据库性能问题

#### 问题症状
- 查询响应缓慢
- 连接池耗尽
- 死锁频繁

#### 诊断步骤

```bash
# 1. 检查慢查询
docker compose exec postgres psql -U postgres -c "
  SELECT query, mean_time, calls 
  FROM pg_stat_statements 
  ORDER BY mean_time DESC 
  LIMIT 10;
"

# 2. 检查锁等待
docker compose exec postgres psql -U postgres -c "
  SELECT * FROM pg_locks WHERE NOT granted;
"

# 3. 分析查询计划
docker compose exec postgres psql -U postgres -c "
  EXPLAIN ANALYZE SELECT * FROM users LIMIT 10;
"
```

#### 解决方案

**1. 索引优化**
```sql
-- 创建必要的索引
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);
CREATE INDEX CONCURRENTLY idx_calls_created_at ON calls(created_at);

-- 分析表统计信息
ANALYZE users;
ANALYZE calls;
```

**2. 连接池调优**
```bash
# 调整连接池参数
export POSTGRES_MAX_CONNECTIONS=200
export DB_POOL_MAX=20
export DB_POOL_IDLE_TIMEOUT=30000

# 重启数据库
docker compose restart postgres
```

## 日志分析

### 日志收集和分析

#### 集中化日志收集

```bash
# 收集所有服务日志
docker compose logs --timestamps > fonoster-logs.txt

# 按服务收集日志
docker compose logs apiserver > apiserver.log
docker compose logs postgres > postgres.log
docker compose logs routr > routr.log

# 实时监控日志
docker compose logs -f --tail=100
```

#### 日志分析脚本

```bash
#!/bin/bash
# analyze-logs.sh - 日志分析脚本

LOG_FILE=${1:-fonoster-logs.txt}

echo "=== 日志分析报告 ==="
echo "日志文件: $LOG_FILE"
echo "分析时间: $(date)"
echo

# 错误统计
echo "=== 错误统计 ==="
grep -i "error" $LOG_FILE | wc -l | xargs echo "错误总数:"
grep -i "fatal" $LOG_FILE | wc -l | xargs echo "致命错误:"
grep -i "warning" $LOG_FILE | wc -l | xargs echo "警告总数:"

# 最常见的错误
echo -e "\n=== 最常见的错误 ==="
grep -i "error" $LOG_FILE | cut -d' ' -f4- | sort | uniq -c | sort -nr | head -10

# 服务重启次数
echo -e "\n=== 服务重启统计 ==="
grep -i "starting\|started" $LOG_FILE | cut -d'|' -f1 | sort | uniq -c

# 性能指标
echo -e "\n=== 性能指标 ==="
grep -i "slow\|timeout\|latency" $LOG_FILE | wc -l | xargs echo "性能问题:"

# 数据库相关问题
echo -e "\n=== 数据库问题 ==="
grep -i "database\|postgres\|connection" $LOG_FILE | grep -i "error\|fail" | wc -l | xargs echo "数据库错误:"

echo -e "\n=== 分析完成 ==="
```

### 常见日志模式

#### API 服务器日志

```bash
# 查找认证失败
grep "authentication failed\|unauthorized" apiserver.log

# 查找数据库连接问题
grep "database connection\|ECONNREFUSED" apiserver.log

# 查找 gRPC 错误
grep "grpc\|rpc error" apiserver.log

# 查找性能问题
grep "slow query\|timeout\|latency" apiserver.log
```

#### Asterisk 日志

```bash
# 查找 SIP 注册问题
docker compose exec asterisk grep "Registration\|REGISTER" /var/log/asterisk/messages

# 查找通话问题
docker compose exec asterisk grep "call\|dial\|hangup" /var/log/asterisk/messages

# 查找音频问题
docker compose exec asterisk grep "rtp\|audio\|codec" /var/log/asterisk/messages
```

#### PostgreSQL 日志

```bash
# 查找连接问题
grep "connection\|authentication" postgres.log

# 查找慢查询
grep "slow query\|duration" postgres.log

# 查找锁等待
grep "deadlock\|lock timeout" postgres.log
```

## 监控和诊断工具

### 系统监控

#### Prometheus + Grafana 配置

```yaml
# monitoring/docker-compose.yml
version: '3.8'
services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-storage:/var/lib/grafana

volumes:
  grafana-storage:
```

```yaml
# monitoring/prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'fonoster-api'
    static_configs:
      - targets: ['apiserver:50051']
    metrics_path: /metrics
    
  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres:5432']
    
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

#### 健康检查端点

```typescript
// health-check.ts
import express from 'express';
import { PrismaClient } from '@prisma/client';

const app = express();
const prisma = new PrismaClient();

app.get('/health', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    services: {}
  };

  try {
    // 检查数据库连接
    await prisma.$queryRaw`SELECT 1`;
    health.services.database = 'ok';
  } catch (error) {
    health.services.database = 'error';
    health.status = 'error';
  }

  // 检查其他服务...
  
  const statusCode = health.status === 'ok' ? 200 : 503;
  res.status(statusCode).json(health);
});

app.get('/metrics', (req, res) => {
  // Prometheus 指标
  const metrics = `
# HELP fonoster_requests_total Total number of requests
# TYPE fonoster_requests_total counter
fonoster_requests_total ${requestCount}

# HELP fonoster_memory_usage Memory usage in bytes
# TYPE fonoster_memory_usage gauge
fonoster_memory_usage ${process.memoryUsage().heapUsed}
  `;
  
  res.set('Content-Type', 'text/plain');
  res.send(metrics);
});
```

### 网络诊断工具

#### SIP 流量分析

```bash
# 使用 tcpdump 捕获 SIP 流量
sudo tcpdump -i any -n -s 0 -w sip-capture.pcap port 5060

# 使用 Wireshark 分析 (GUI)
wireshark sip-capture.pcap

# 使用 tshark 命令行分析
tshark -r sip-capture.pcap -Y sip -T fields -e sip.Method -e sip.Status-Code
```

#### RTP 流量监控

```bash
# 监控 RTP 流量
sudo tcpdump -i any -n udp portrange 10000-20000

# 分析 RTP 统计
docker compose exec asterisk asterisk -rx "rtp show stats"

# 检查 RTP 质量
docker compose exec asterisk asterisk -rx "rtp show quality"
```

### 性能分析工具

#### Node.js 性能分析

```bash
# 启用性能分析
docker compose exec apiserver node --prof --prof-process app.js

# 生成火焰图
docker compose exec apiserver node --perf-basic-prof app.js &
sudo perf record -F 99 -p $! -g -- sleep 30
sudo perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg
```

#### 数据库性能分析

```sql
-- 启用查询统计
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 查看最慢的查询
SELECT 
  query,
  calls,
  total_time,
  mean_time,
  rows
FROM pg_stat_statements 
ORDER BY mean_time DESC 
LIMIT 10;

-- 查看最频繁的查询
SELECT 
  query,
  calls,
  total_time,
  mean_time
FROM pg_stat_statements 
ORDER BY calls DESC 
LIMIT 10;
```

## 常见错误代码

### HTTP 错误代码

| 错误代码 | 含义 | 常见原因 | 解决方案 |
|---------|------|----------|----------|
| 400 | 请求错误 | 参数格式错误 | 检查请求参数格式 |
| 401 | 未授权 | 认证失败 | 检查 API 密钥或 JWT 令牌 |
| 403 | 禁止访问 | 权限不足 | 检查用户权限配置 |
| 404 | 资源不存在 | 路径错误 | 检查 API 路径 |
| 429 | 请求过多 | 触发限流 | 降低请求频率 |
| 500 | 服务器错误 | 内部错误 | 检查服务器日志 |
| 502 | 网关错误 | 上游服务不可用 | 检查后端服务状态 |
| 503 | 服务不可用 | 服务过载 | 检查系统资源 |
| 504 | 网关超时 | 请求超时 | 增加超时时间 |

### gRPC 错误代码

| 错误代码 | 含义 | 常见原因 | 解决方案 |
|---------|------|----------|----------|
| CANCELLED | 请求被取消 | 客户端取消 | 检查客户端逻辑 |
| UNKNOWN | 未知错误 | 服务器异常 | 检查服务器日志 |
| INVALID_ARGUMENT | 参数无效 | 参数格式错误 | 验证请求参数 |
| DEADLINE_EXCEEDED | 超时 | 请求超时 | 增加超时时间 |
| NOT_FOUND | 资源不存在 | 资源不存在 | 检查资源 ID |
| ALREADY_EXISTS | 资源已存在 | 重复创建 | 检查唯一性约束 |
| PERMISSION_DENIED | 权限被拒绝 | 权限不足 | 检查权限配置 |
| UNAUTHENTICATED | 未认证 | 认证失败 | 检查认证信息 |
| RESOURCE_EXHAUSTED | 资源耗尽 | 限流或资源不足 | 检查系统资源 |
| FAILED_PRECONDITION | 前置条件失败 | 状态不正确 | 检查业务逻辑 |
| ABORTED | 操作中止 | 冲突或事务失败 | 重试操作 |
| OUT_OF_RANGE | 超出范围 | 参数超出范围 | 检查参数范围 |
| UNIMPLEMENTED | 未实现 | 功能未实现 | 检查 API 版本 |
| INTERNAL | 内部错误 | 服务器内部错误 | 检查服务器日志 |
| UNAVAILABLE | 服务不可用 | 服务不可用 | 检查服务状态 |
| DATA_LOSS | 数据丢失 | 数据损坏 | 检查数据完整性 |

### SIP 错误代码

| 错误代码 | 含义 | 常见原因 | 解决方案 |
|---------|------|----------|----------|
| 400 | 请求错误 | SIP 消息格式错误 | 检查 SIP 消息格式 |
| 401 | 未授权 | 认证失败 | 检查 SIP 认证信息 |
| 403 | 禁止 | 权限不足 | 检查 SIP 权限配置 |
| 404 | 用户不存在 | 用户未注册 | 检查用户注册状态 |
| 408 | 请求超时 | 网络超时 | 检查网络连接 |
| 480 | 暂时不可用 | 用户离线 | 检查用户状态 |
| 486 | 忙线 | 用户忙线 | 稍后重试 |
| 487 | 请求终止 | 请求被取消 | 检查取消原因 |
| 500 | 服务器内部错误 | 服务器错误 | 检查服务器日志 |
| 503 | 服务不可用 | 服务过载 | 检查服务器负载 |

## 紧急恢复程序

### 服务完全不可用

#### 紧急恢复步骤

```bash
#!/bin/bash
# emergency-recovery.sh - 紧急恢复脚本

echo "=== 开始紧急恢复程序 ==="

# 1. 停止所有服务
echo "1. 停止所有服务..."
docker compose down

# 2. 清理资源
echo "2. 清理 Docker 资源..."
docker system prune -f
docker volume prune -f

# 3. 检查磁盘空间
echo "3. 检查磁盘空间..."
df -h
if [ $(df / | tail -1 | awk '{print $5}' | sed 's/%//') -gt 90 ]; then
  echo "警告: 磁盘空间不足!"
  # 清理日志文件
  find /var/log -name "*.log" -mtime +7 -delete
  # 清理临时文件
  rm -rf /tmp/*
fi

# 4. 备份配置
echo "4. 备份当前配置..."
cp .env .env.backup.$(date +%Y%m%d_%H%M%S)

# 5. 重置配置
echo "5. 重置配置..."
cp .env.example .env

# 6. 生成新密钥
echo "6. 生成新密钥..."
npm run generate:keypair

# 7. 重新启动服务
echo "7. 重新启动服务..."
docker compose up -d

# 8. 等待服务就绪
echo "8. 等待服务就绪..."
sleep 60

# 9. 健康检查
echo "9. 执行健康检查..."
./scripts/health-check.sh

echo "=== 紧急恢复完成 ==="
```

### 数据库恢复

#### 数据库备份恢复

```bash
#!/bin/bash
# restore-database.sh - 数据库恢复脚本

BACKUP_FILE=${1:-latest}
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

echo "=== 数据库恢复程序 ==="

# 1. 停止 API 服务器
echo "1. 停止 API 服务器..."
docker compose stop apiserver

# 2. 备份当前数据库
echo "2. 备份当前数据库..."
docker compose exec postgres pg_dump -U postgres fonoster > backup_before_restore_$TIMESTAMP.sql

# 3. 恢复数据库
echo "3. 恢复数据库..."
if [ "$BACKUP_FILE" = "latest" ]; then
  BACKUP_FILE=$(ls -t backup_*.sql | head -1)
fi

docker compose exec -T postgres psql -U postgres -c "DROP DATABASE IF EXISTS fonoster;"
docker compose exec -T postgres psql -U postgres -c "CREATE DATABASE fonoster;"
docker compose exec -T postgres psql -U postgres fonoster < $BACKUP_FILE

# 4. 运行迁移
echo "4. 运行数据库迁移..."
cd mods/apiserver
npx prisma migrate deploy
cd ../..

# 5. 重启服务
echo "5. 重启服务..."
docker compose start apiserver

echo "=== 数据库恢复完成 ==="
```

### 配置回滚

#### 配置版本管理

```bash
#!/bin/bash
# config-rollback.sh - 配置回滚脚本

VERSION=${1:-previous}

echo "=== 配置回滚程序 ==="

# 1. 列出可用的配置备份
echo "可用的配置备份:"
ls -la .env.backup.*

# 2. 选择回滚版本
if [ "$VERSION" = "previous" ]; then
  BACKUP_FILE=$(ls -t .env.backup.* | head -1)
else
  BACKUP_FILE=".env.backup.$VERSION"
fi

if [ ! -f "$BACKUP_FILE" ]; then
  echo "错误: 备份文件 $BACKUP_FILE 不存在"
  exit 1
fi

# 3. 备份当前配置
echo "备份当前配置..."
cp .env .env.backup.$(date +%Y%m%d_%H%M%S)

# 4. 恢复配置
echo "恢复配置: $BACKUP_FILE"
cp $BACKUP_FILE .env

# 5. 重启服务
echo "重启服务..."
docker compose restart

echo "=== 配置回滚完成 ==="
```

### 灾难恢复计划

#### 完整系统恢复

```bash
#!/bin/bash
# disaster-recovery.sh - 灾难恢复脚本

BACKUP_DIR=${1:-/backup/fonoster}
RECOVERY_DATE=${2:-$(date +%Y%m%d)}

echo "=== 灾难恢复程序 ==="
echo "备份目录: $BACKUP_DIR"
echo "恢复日期: $RECOVERY_DATE"

# 1. 验证备份文件
echo "1. 验证备份文件..."
if [ ! -d "$BACKUP_DIR/$RECOVERY_DATE" ]; then
  echo "错误: 备份目录不存在"
  exit 1
fi

# 2. 停止所有服务
echo "2. 停止所有服务..."
docker compose down -v

# 3. 恢复配置文件
echo "3. 恢复配置文件..."
cp $BACKUP_DIR/$RECOVERY_DATE/.env .env
cp -r $BACKUP_DIR/$RECOVERY_DATE/.keys .keys

# 4. 恢复数据库
echo "4. 恢复数据库..."
docker compose up -d postgres
sleep 30
docker compose exec -T postgres psql -U postgres < $BACKUP_DIR/$RECOVERY_DATE/database.sql

# 5. 恢复文件存储
echo "5. 恢复文件存储..."
if [ -d "$BACKUP_DIR/$RECOVERY_DATE/storage" ]; then
  cp -r $BACKUP_DIR/$RECOVERY_DATE/storage/* /var/lib/fonoster/storage/
fi

# 6. 启动所有服务
echo "6. 启动所有服务..."
docker compose up -d

# 7. 验证恢复
echo "7. 验证恢复..."
sleep 60
./scripts/health-check.sh

echo "=== 灾难恢复完成 ==="
```

---

## 获取帮助

如果本指南无法解决你遇到的问题，请通过以下方式获取帮助：

### 社区支持
- **Discord**: [加入 Fonoster Discord 社区](https://discord.gg/4QWgSz4hTC)
- **GitHub Issues**: [报告问题](https://github.com/fonoster/fonoster/issues)
- **GitHub Discussions**: [参与讨论](https://github.com/fonoster/fonoster/discussions)

### 商业支持
- **企业支持**: 联系 [support@fonoster.com](mailto:support@fonoster.com)
- **专业服务**: 访问 [https://fonoster.com/services](https://fonoster.com/services)

### 提交问题时请包含

1. **环境信息**: 操作系统、Docker 版本、Node.js 版本
2. **错误描述**: 详细的错误信息和重现步骤
3. **日志文件**: 相关的日志输出
4. **配置信息**: 脱敏后的配置文件
5. **系统状态**: `docker compose ps` 和 `docker stats` 输出

---

**记住**: 大多数问题都可以通过仔细阅读日志和系统状态来诊断。保持冷静，系统性地排查问题，你一定能找到解决方案！ 🔧