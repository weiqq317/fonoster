# API Server 模块文档

## 概述

API Server 是 Fonoster 的核心服务模块，提供统一的 gRPC 和 HTTP 接口，负责协调所有其他服务模块的交互。它是整个系统的中央枢纽，处理客户端请求、业务逻辑和数据持久化。

## 核心功能

### 1. 服务编排
- 统一的 gRPC 服务接口
- HTTP 桥接支持
- 服务间通信协调
- 健康检查和监控

### 2. 身份认证和授权
- JWT 令牌验证
- OAuth2 集成
- 基于角色的访问控制 (RBAC)
- API 密钥管理

### 3. 通话管理
- 通话创建和控制
- 语音应用集成
- Asterisk ARI 接口
- 实时通话事件处理

### 4. 数据管理
- Prisma ORM 数据访问
- PostgreSQL 数据库集成
- 数据加密和安全
- 事务管理

## 架构设计

### 目录结构

```
mods/apiserver/
├── src/
│   ├── applications/        # 应用管理服务
│   ├── calls/              # 通话管理服务
│   ├── core/               # 核心功能和配置
│   ├── events/             # 事件处理
│   ├── secrets/            # 密钥管理
│   ├── utils/              # 工具函数
│   ├── voice/              # 语音处理
│   ├── envs.ts             # 环境变量配置
│   └── index.ts            # 入口文件
├── migrations/             # 数据库迁移文件
├── schema.prisma           # Prisma 数据模型
├── package.json
└── tsconfig.json
```

### 核心组件

#### 1. 服务启动器 (`core/runServices.ts`)

负责启动和配置所有服务：

```typescript
async function runServices() {
  // 创建健康检查服务
  const healthImpl = new HealthImplementation(statusMap);
  
  // 配置认证拦截器
  const authorization = createAuthInterceptor(IDENTITY_PUBLIC_KEY, allowList);
  const checkMethodAuthorized = createCheckMethodAuthorized(
    `${AUTHZ_SERVICE_HOST}:${AUTHZ_SERVICE_PORT}`,
    AUTHZ_SERVICE_METHODS
  );
  
  // 创建 gRPC 服务器
  const server = new grpc.Server({ interceptors });
  
  // 加载所有服务
  loadServices(server, await services);
  
  // 连接到 Asterisk ARI
  await connectToAri(httpBridge(identityConfig, { port: HTTP_BRIDGE_PORT }));
  
  // 启动通话管理器
  await runCallManager({
    natsUrl: NATS_URL,
    ariProxyUrl: ASTERISK_ARI_PROXY_URL,
    ariUsername: ASTERISK_ARI_USERNAME,
    ariPassword: ASTERISK_ARI_SECRET
  });
  
  // 绑定服务器地址
  server.bindAsync(APISERVER_BIND_ADDR, credentials, async () => {
    healthImpl.setStatus("", GRPC_SERVING_STATUS);
    logger.info(`apiserver running at ${APISERVER_BIND_ADDR}`);
  });
}
```

#### 2. 服务定义 (`core/services.ts`)

定义所有可用的服务：

```typescript
const services = [
  identityService,           // 身份认证服务
  applicationsService,       // 应用管理服务
  callsService,             // 通话管理服务
  secretsService,           // 密钥管理服务
  aclsService,              // 访问控制列表服务
  agentsService,            // 代理服务
  credentialsService,       // 凭证服务
  domainsService,           // 域名服务
  numbersService,           // 号码服务
  trunksService,            // 中继服务
  welcomeDemoService        // 欢迎演示服务
];
```

#### 3. 语音客户端 (`voice/VoiceClientImpl.ts`)

处理语音应用的核心逻辑：

```typescript
class VoiceClientImpl implements VoiceClient {
  private config: VoiceClientConfig;
  private verbsStream: Stream;
  private transcriptionsStream: Stream;
  private tts: TextToSpeech;
  private stt: SpeechToText;
  private ari: Client;
  private filesServer;

  constructor(params: {
    ari: Client;
    config: VoiceClientConfig;
    tts: TextToSpeech;
    stt: SpeechToText;
  }, filesServer) {
    // 初始化语音客户端
  }

  // 实现语音指令处理方法
  async answer(): Promise<void> { /* ... */ }
  async hangup(): Promise<void> { /* ... */ }
  async play(url: string): Promise<void> { /* ... */ }
  async say(text: string): Promise<void> { /* ... */ }
  async gather(options: GatherOptions): Promise<GatherResult> { /* ... */ }
  // ... 其他语音指令
}
```

## 主要服务

### 1. 应用管理服务 (`applications/`)

管理语音应用的生命周期：

- **创建应用**: 注册新的语音应用
- **更新应用**: 修改应用配置
- **删除应用**: 移除应用
- **列表查询**: 获取应用列表
- **应用验证**: 验证应用配置的有效性

### 2. 通话管理服务 (`calls/`)

处理通话相关的所有操作：

- **创建通话**: 发起新的通话
- **通话控制**: 控制通话状态
- **通话记录**: 记录通话详情
- **事件处理**: 处理通话事件
- **统计分析**: 通话数据分析

### 3. 密钥管理服务 (`secrets/`)

管理系统密钥和敏感信息：

- **密钥存储**: 安全存储敏感信息
- **密钥检索**: 获取密钥信息
- **访问控制**: 控制密钥访问权限
- **加密解密**: 数据加密和解密

## 数据模型

### Prisma Schema 主要模型

```prisma
model Application {
  ref           String            @id @default(cuid())
  name          String
  type          ApplicationType
  endpoint      String?
  textToSpeech  Json?
  speechToText  Json?
  intelligence  Json?
  accessKeyId   String
  createdAt     DateTime          @default(now())
  updatedAt     DateTime          @updatedAt
  
  @@map("applications")
}

model Secret {
  ref         String   @id @default(cuid())
  name        String
  secret      String   @encrypted
  accessKeyId String
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@map("secrets")
}

model CallDetailRecord {
  ref           String    @id @default(cuid())
  accessKeyId   String
  appRef        String?
  sessionRef    String    @unique
  from          String
  to            String
  direction     Direction
  status        CallStatus
  startTime     DateTime?
  answerTime    DateTime?
  endTime       DateTime?
  duration      Int?      @default(0)
  billableTime  Int?      @default(0)
  
  @@map("call_detail_records")
}
```

## 配置管理

### 环境变量

API Server 支持丰富的环境变量配置：

```typescript
// 服务绑定配置
export const APISERVER_BIND_ADDR = "0.0.0.0:50051";
export const APISERVER_HOST = "apiserver";

// Asterisk 配置
export const ASTERISK_ARI_PROXY_URL = process.env.APISERVER_ASTERISK_ARI_PROXY_URL;
export const ASTERISK_ARI_SECRET = process.env.APISERVER_ASTERISK_ARI_SECRET;
export const ASTERISK_ARI_USERNAME = process.env.APISERVER_ASTERISK_ARI_USERNAME;

// 数据库配置
export const DATABASE_URL = process.env.APISERVER_DATABASE_URL;
export const IDENTITY_DATABASE_URL = process.env.APISERVER_IDENTITY_DATABASE_URL;

// 消息队列配置
export const NATS_URL = process.env.APISERVER_NATS_URL;

// 身份认证配置
export const IDENTITY_ISSUER = process.env.APISERVER_IDENTITY_ISSUER;
export const IDENTITY_PUBLIC_KEY_PATH = "/opt/fonoster/keys/public.pem";
export const IDENTITY_PRIVATE_KEY_PATH = "/opt/fonoster/keys/private.pem";

// 授权服务配置
export const AUTHZ_SERVICE_ENABLED = process.env.APISERVER_AUTHZ_SERVICE_ENABLED === "true";
export const AUTHZ_SERVICE_HOST = process.env.APISERVER_AUTHZ_SERVICE_HOST;
export const AUTHZ_SERVICE_PORT = process.env.APISERVER_AUTHZ_SERVICE_PORT;
```

### 身份认证配置

```typescript
const identityConfig = {
  issuer: IDENTITY_ISSUER,
  publicKeyPath: IDENTITY_PUBLIC_KEY_PATH,
  privateKeyPath: IDENTITY_PRIVATE_KEY_PATH,
  accessTokenExpiresIn: IDENTITY_ACCESS_TOKEN_EXPIRES_IN,
  refreshTokenExpiresIn: IDENTITY_REFRESH_TOKEN_EXPIRES_IN
};
```

## 集成接口

### 1. Asterisk ARI 集成

API Server 通过 ARI (Asterisk REST Interface) 与 Asterisk 媒体服务器集成：

```typescript
async function connectToAri(filesServer) {
  const ari = await ariClient.connect(
    ASTERISK_ARI_PROXY_URL,
    ASTERISK_ARI_USERNAME,
    ASTERISK_ARI_SECRET
  );

  // 处理重连事件
  ari.on(AriEvent.WEB_SOCKET_RECONNECTING, () => {
    logger.info("reconnecting to asterisk");
  });

  // 创建语音调度器
  const dispatcher = new VoiceDispatcher(
    ari,
    nats,
    createCreateVoiceClient(createContainer, filesServer)
  );

  dispatcher.start();
}
```

### 2. NATS 消息队列集成

使用 NATS 进行服务间异步通信：

```typescript
const nats = await connect({ 
  servers: NATS_URL, 
  maxReconnectAttempts: -1 
});

// 订阅通话创建事件
const subscription = nats.subscribe(CALLS_CREATE_SUBJECT, {
  queue: DEFAULT_NATS_QUEUE_GROUP
});
```

### 3. Routr SIP 服务器集成

通过 Routr SDK 管理 SIP 网络：

```typescript
const routrConfig = {
  endpoint: ROUTR_API_ENDPOINT,
  insecure: true
};

const routr = new SDK(routrConfig);
```

## 安全特性

### 1. 认证拦截器

所有 gRPC 请求都经过认证拦截器验证：

```typescript
const authorization = createAuthInterceptor(IDENTITY_PUBLIC_KEY, allowList);
```

### 2. 授权检查

支持细粒度的方法级授权：

```typescript
const checkMethodAuthorized = createCheckMethodAuthorized(
  `${AUTHZ_SERVICE_HOST}:${AUTHZ_SERVICE_PORT}`,
  AUTHZ_SERVICE_METHODS
);
```

### 3. 数据加密

敏感数据使用 Prisma 字段级加密：

```prisma
model Secret {
  secret String @encrypted
}
```

## 监控和日志

### 1. 健康检查

提供标准的 gRPC 健康检查服务：

```typescript
const healthImpl = new HealthImplementation(statusMap);
healthImpl.addToServer(server);
healthImpl.setStatus("", GRPC_SERVING_STATUS);
```

### 2. 结构化日志

使用统一的日志格式：

```typescript
const logger = getLogger({ service: "apiserver", filePath: __filename });
logger.info("apiserver running", { address: APISERVER_BIND_ADDR });
```

### 3. 指标收集

集成 InfluxDB 进行指标收集和分析：

```typescript
const influxdb = new InfluxDB({
  url: INFLUXDB_URL,
  token: INFLUXDB_INIT_TOKEN
});
```

## 开发指南

### 1. 本地开发

```bash
# 启动开发环境
npm run start:apiserver

# 启动依赖服务
npm run start:services
```

### 2. 数据库迁移

```bash
# 生成迁移文件
npx prisma migrate dev --name migration_name

# 应用迁移
npx prisma migrate deploy

# 生成 Prisma 客户端
npx prisma generate
```

### 3. 添加新服务

1. 在相应目录下创建服务文件
2. 实现服务接口
3. 在 `core/services.ts` 中注册服务
4. 更新 proto 文件定义
5. 添加相应的测试

### 4. 调试技巧

```bash
# 启用详细日志
export LOGS_LEVEL=verbose

# 启用开发模式
export NODE_ENV=development

# 查看 gRPC 调用
export GRPC_VERBOSITY=DEBUG
```

## 故障排除

### 常见问题

1. **服务启动失败**
   - 检查环境变量配置
   - 确认数据库连接
   - 验证依赖服务状态

2. **认证失败**
   - 检查 JWT 密钥配置
   - 验证令牌有效性
   - 确认权限设置

3. **Asterisk 连接问题**
   - 检查 ARI 配置
   - 验证网络连接
   - 查看 Asterisk 日志

4. **数据库连接问题**
   - 检查连接字符串
   - 验证数据库状态
   - 确认迁移状态

### 日志分析

```bash
# 查看服务日志
docker logs fonoster-apiserver

# 过滤错误日志
docker logs fonoster-apiserver 2>&1 | grep ERROR

# 实时监控日志
docker logs -f fonoster-apiserver
```

## 性能优化

### 1. 数据库优化

- 使用连接池
- 添加适当索引
- 优化查询语句
- 实施数据分页

### 2. 缓存策略

- Redis 缓存热点数据
- 内存缓存频繁访问的配置
- CDN 缓存静态资源

### 3. 并发处理

- 异步处理长时间操作
- 使用消息队列解耦
- 实施背压控制

## 扩展指南

### 1. 添加新的语音指令

1. 在 `voice/types.ts` 中定义新指令类型
2. 在 `VoiceClientImpl.ts` 中实现指令逻辑
3. 更新 proto 文件定义
4. 添加相应测试

### 2. 集成新的 TTS/STT 引擎

1. 在 `voice/tts/` 或 `voice/stt/` 目录下创建新引擎
2. 实现相应接口
3. 在配置中注册新引擎
4. 添加配置选项

### 3. 添加新的存储后端

1. 实现存储接口
2. 添加配置选项
3. 更新文件服务
4. 添加迁移脚本

这个 API Server 模块是 Fonoster 系统的核心，理解其架构和工作原理对于系统的开发、部署和维护都至关重要。