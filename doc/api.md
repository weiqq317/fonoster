# Fonoster API 接口文档

## 概述

Fonoster 提供了完整的 RESTful API 和 gRPC 接口，用于构建和管理语音通信应用。本文档详细介绍了所有可用的 API 端点、认证方式、请求/响应格式以及使用示例。

## 基础信息

### API 端点

- **生产环境**: `https://api.fonoster.com`
- **开发环境**: `http://localhost:50051` (gRPC) / `http://localhost:8080` (HTTP)

### 协议支持

- **gRPC**: 高性能的二进制协议，适用于服务间通信
- **gRPC-Web**: 浏览器兼容的 gRPC 实现
- **HTTP/REST**: 传统的 RESTful API，通过 Envoy 代理提供

### 版本控制

当前 API 版本：`v1beta2`

## 认证方式

### 1. API 密钥认证（推荐）

```javascript
const SDK = require("@fonoster/sdk");

const client = new SDK.Client({ 
  accessKeyId: "WO00000000000000000000000000000000",
  endpoint: "api.fonoster.com" 
});

await client.loginWithApiKey("your-api-key", "your-api-secret");
```

### 2. 用户名密码认证

```javascript
const client = new SDK.Client({ 
  accessKeyId: "WO00000000000000000000000000000000" 
});

await client.login("admin@fonoster.local", "changeme");
```

### 3. 刷新令牌认证

```javascript
await client.loginWithRefreshToken("your-refresh-token");
```

### 认证头格式

```http
Authorization: Bearer <access_token>
AccessKeyId: WO00000000000000000000000000000000
```

## 核心 API 服务

### 1. Applications API

管理语音应用的生命周期。

#### 创建应用

```http
POST /v1beta2/applications
Content-Type: application/json
Authorization: Bearer <token>
AccessKeyId: <workspace_id>

{
  "name": "My Voice App",
  "type": "EXTERNAL",
  "endpoint": "https://myapp.example.com/webhook",
  "textToSpeech": {
    "productRef": "tts-google",
    "config": {
      "voice": "en-US-Wavenet-D"
    }
  },
  "speechToText": {
    "productRef": "stt-google",
    "config": {
      "languageCode": "en-US"
    }
  },
  "intelligence": {
    "productRef": "openai-gpt4",
    "credentials": {
      "apiKey": "sk-..."
    },
    "config": {
      "model": "gpt-4",
      "temperature": 0.7
    }
  }
}
```

**响应**:
```json
{
  "ref": "3e61ecb7-a1b6-4a93-84c3-4f1979165bca",
  "name": "My Voice App",
  "type": "EXTERNAL",
  "endpoint": "https://myapp.example.com/webhook",
  "createdAt": "2025-01-01T00:00:00Z",
  "updatedAt": "2025-01-01T00:00:00Z"
}
```

#### 获取应用列表

```http
GET /v1beta2/applications?pageSize=20&pageToken=<token>
Authorization: Bearer <token>
AccessKeyId: <workspace_id>
```

**响应**:
```json
{
  "items": [
    {
      "ref": "3e61ecb7-a1b6-4a93-84c3-4f1979165bca",
      "name": "My Voice App",
      "type": "EXTERNAL",
      "endpoint": "https://myapp.example.com/webhook"
    }
  ],
  "nextPageToken": "next_page_token"
}
```

#### 更新应用

```http
PUT /v1beta2/applications/{ref}
Content-Type: application/json

{
  "name": "Updated App Name",
  "endpoint": "https://newapp.example.com/webhook"
}
```

#### 删除应用

```http
DELETE /v1beta2/applications/{ref}
```

#### 创建测试令牌

```http
POST /v1beta2/applications/{ref}/test-token
```

**响应**:
```json
{
  "domain": "sip.fonoster.local",
  "displayName": "Test User",
  "signalingServer": "wss://sip.fonoster.local",
  "targetAor": "sip:test@sip.fonoster.local",
  "username": "test",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### 2. Calls API

管理通话会话和通话记录。

#### 创建通话

```http
POST /v1beta2/calls
Content-Type: application/json

{
  "from": "+18287854037",
  "to": "+17853178070",
  "appRef": "3e61ecb7-a1b6-4a93-84c3-4f1979165bca"
}
```

**响应**:
```json
{
  "ref": "call-12345",
  "status": "TRYING"
}
```

#### 获取通话列表

```http
GET /v1beta2/calls?pageSize=20&pageToken=<token>
```

**响应**:
```json
{
  "items": [
    {
      "ref": "call-12345",
      "from": "+18287854037",
      "to": "+17853178070",
      "status": "NORMAL_CLEARING",
      "direction": "TO_PSTN",
      "type": "API_ORIGINATED",
      "startTime": "2025-01-01T10:00:00Z",
      "endTime": "2025-01-01T10:05:30Z",
      "duration": 330
    }
  ],
  "nextPageToken": "next_page_token"
}
```

#### 获取通话详情

```http
GET /v1beta2/calls/{ref}
```

#### 跟踪通话状态（流式）

```javascript
const calls = new SDK.Calls(client);
const stream = calls.trackCall({ ref: "call-12345" });

stream.on('data', (response) => {
  console.log('Call status:', response.status);
});
```

### 3. Numbers API

管理电话号码资源。

#### 创建号码

```http
POST /v1beta2/numbers
Content-Type: application/json

{
  "name": "Main Office Line",
  "telUrl": "tel:+17853178070",
  "city": "Asheville",
  "country": "United States",
  "countryIsoCode": "US",
  "ingressHandler": {
    "type": "AGENT",
    "ref": "agent-ref"
  }
}
```

#### 获取号码列表

```http
GET /v1beta2/numbers
```

#### 更新号码

```http
PUT /v1beta2/numbers/{ref}
Content-Type: application/json

{
  "name": "Updated Number Name",
  "ingressHandler": {
    "type": "APPLICATION",
    "ref": "app-ref"
  }
}
```

### 4. Agents API

管理 SIP 代理（用户/设备）。

#### 创建代理

```http
POST /v1beta2/agents
Content-Type: application/json

{
  "name": "John Doe",
  "username": "john.doe",
  "domain": {
    "ref": "domain-ref"
  },
  "credentials": {
    "ref": "credentials-ref"
  },
  "privacy": "PRIVATE",
  "enabled": true
}
```

#### 获取代理列表

```http
GET /v1beta2/agents
```

### 5. Domains API

管理 SIP 域。

#### 创建域

```http
POST /v1beta2/domains
Content-Type: application/json

{
  "name": "My SIP Domain",
  "domainUri": "sip.example.com",
  "accessControlList": {
    "ref": "acl-ref"
  },
  "egressPolicies": [
    {
      "rule": ".*",
      "numberRef": "number-ref"
    }
  ]
}
```

### 6. Trunks API

管理 SIP 中继。

#### 创建中继

```http
POST /v1beta2/trunks
Content-Type: application/json

{
  "name": "Main Trunk",
  "inboundUri": "sip:trunk@provider.com",
  "outboundUri": "sip:provider.com",
  "accessControlList": {
    "ref": "acl-ref"
  },
  "inboundCredentials": {
    "ref": "credentials-ref"
  },
  "outboundCredentials": {
    "ref": "credentials-ref"
  }
}
```

### 7. Credentials API

管理 SIP 认证凭据。

#### 创建凭据

```http
POST /v1beta2/credentials
Content-Type: application/json

{
  "name": "SIP Credentials",
  "username": "sipuser",
  "password": "securepassword"
}
```

### 8. ACLs API

管理访问控制列表。

#### 创建 ACL

```http
POST /v1beta2/acls
Content-Type: application/json

{
  "name": "Office Network",
  "allow": ["192.168.1.0/24", "10.0.0.0/8"],
  "deny": ["192.168.1.100"]
}
```

### 9. Secrets API

管理敏感配置信息。

#### 创建密钥

```http
POST /v1beta2/secrets
Content-Type: application/json

{
  "name": "API_KEY",
  "secret": "encrypted_secret_value"
}
```

### 10. Identity API

管理工作空间、用户和认证。

#### 创建工作空间

```http
POST /v1beta2/workspaces
Content-Type: application/json

{
  "name": "My Company",
  "ownerEmail": "admin@company.com"
}
```

#### 创建用户

```http
POST /v1beta2/users
Content-Type: application/json

{
  "name": "John Doe",
  "email": "john@company.com",
  "password": "securepassword",
  "avatar": "https://example.com/avatar.jpg"
}
```

#### 创建 API 密钥

```http
POST /v1beta2/apikeys
Content-Type: application/json

{
  "role": "WORKSPACE_ADMIN"
}
```

**响应**:
```json
{
  "ref": "apikey-ref",
  "accessKeyId": "AK1234567890",
  "accessKeySecret": "SK1234567890abcdef",
  "role": "WORKSPACE_ADMIN"
}
```

## 错误处理

### HTTP 状态码

- `200` - 成功
- `201` - 创建成功
- `400` - 请求参数错误
- `401` - 未认证
- `403` - 权限不足
- `404` - 资源不存在
- `409` - 资源冲突
- `500` - 服务器内部错误

### gRPC 状态码

- `OK (0)` - 成功
- `INVALID_ARGUMENT (3)` - 参数无效
- `UNAUTHENTICATED (16)` - 未认证
- `PERMISSION_DENIED (7)` - 权限不足
- `NOT_FOUND (5)` - 资源不存在
- `ALREADY_EXISTS (6)` - 资源已存在
- `INTERNAL (13)` - 内部错误

### 错误响应格式

```json
{
  "error": {
    "code": "INVALID_ARGUMENT",
    "message": "The request has one or more invalid arguments",
    "details": [
      {
        "field": "name",
        "message": "Name is required"
      }
    ]
  }
}
```

## SDK 使用指南

### Node.js SDK

#### 安装

```bash
npm install @fonoster/sdk
```

#### 基本使用

```javascript
const SDK = require("@fonoster/sdk");

async function main() {
  // 创建客户端
  const client = new SDK.Client({ 
    accessKeyId: "WO00000000000000000000000000000000" 
  });

  // 认证
  await client.loginWithApiKey("your-api-key", "your-api-secret");

  // 使用服务
  const applications = new SDK.Applications(client);
  const agents = new SDK.Agents(client);
  const calls = new SDK.Calls(client);
  const numbers = new SDK.Numbers(client);

  // 创建应用
  const app = await applications.createApplication({
    name: "My App",
    type: "EXTERNAL",
    endpoint: "https://myapp.com/webhook"
  });

  // 发起通话
  const call = await calls.createCall({
    from: "+1234567890",
    to: "+0987654321",
    appRef: app.ref
  });

  console.log("Call created:", call.ref);
}

main().catch(console.error);
```

### 浏览器 SDK

#### 安装

```html
<script src="https://unpkg.com/@fonoster/sdk"></script>
```

或使用 npm:

```bash
npm install @fonoster/sdk
```

#### 基本使用

```javascript
import * as SDK from "@fonoster/sdk";

async function main() {
  // 创建 Web 客户端
  const client = new SDK.WebClient({ 
    accessKeyId: "WO00000000000000000000000000000000",
    url: "https://api.fonoster.com/v1beta2"
  });

  // 认证
  await client.login("user@example.com", "password");

  // 使用服务
  const applications = new SDK.Applications(client);
  
  const apps = await applications.listApplications({
    pageSize: 10
  });

  console.log("Applications:", apps.items);
}
```

## 语音应用开发

### Webhook 接口

当有来电时，Fonoster 会向您的应用端点发送 POST 请求：

```http
POST /webhook
Content-Type: application/json

{
  "sessionRef": "session-12345",
  "callerNumber": "+1234567890",
  "calleeNumber": "+0987654321",
  "domain": "sip.fonoster.local"
}
```

### 语音指令

您的应用需要返回语音指令来控制通话：

```json
{
  "commands": [
    {
      "say": {
        "text": "Hello, welcome to our service!"
      }
    },
    {
      "gather": {
        "source": "speech",
        "timeout": 5000,
        "finishOnKey": "#"
      }
    },
    {
      "hangup": {}
    }
  ]
}
```

### 支持的指令

#### Say - 文本转语音

```json
{
  "say": {
    "text": "Hello world",
    "voice": "en-US-Wavenet-D"
  }
}
```

#### Play - 播放音频

```json
{
  "play": {
    "url": "https://example.com/audio.wav"
  }
}
```

#### Gather - 收集输入

```json
{
  "gather": {
    "source": "speech",
    "timeout": 5000,
    "finishOnKey": "#",
    "hints": ["yes", "no", "help"]
  }
}
```

#### Record - 录音

```json
{
  "record": {
    "timeout": 30000,
    "finishOnKey": "#",
    "maxDuration": 60000
  }
}
```

#### Dial - 转接通话

```json
{
  "dial": {
    "destination": "+1234567890",
    "timeout": 30000
  }
}
```

#### Hangup - 挂断

```json
{
  "hangup": {}
}
```

## 实时事件

### WebSocket 连接

```javascript
const ws = new WebSocket('wss://api.fonoster.com/events');

ws.onopen = () => {
  // 订阅事件
  ws.send(JSON.stringify({
    type: 'subscribe',
    events: ['call.created', 'call.answered', 'call.ended']
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Event received:', data);
};
```

### 事件类型

- `call.created` - 通话创建
- `call.answered` - 通话接听
- `call.ended` - 通话结束
- `agent.registered` - 代理注册
- `agent.unregistered` - 代理注销

## 最佳实践

### 1. 错误处理

```javascript
try {
  const result = await api.someMethod(request);
  return result;
} catch (error) {
  if (error.code === 'UNAUTHENTICATED') {
    // 重新认证
    await client.refreshToken();
    return await api.someMethod(request);
  }
  throw error;
}
```

### 2. 分页处理

```javascript
async function getAllItems(listMethod) {
  const items = [];
  let pageToken = '';
  
  do {
    const response = await listMethod({
      pageSize: 100,
      pageToken
    });
    
    items.push(...response.items);
    pageToken = response.nextPageToken;
  } while (pageToken);
  
  return items;
}
```

### 3. 并发控制

```javascript
const pLimit = require('p-limit');
const limit = pLimit(5); // 最多 5 个并发请求

const promises = items.map(item => 
  limit(() => api.processItem(item))
);

const results = await Promise.all(promises);
```

### 4. 重试机制

```javascript
async function withRetry(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

## 限制和配额

### API 限制

- **请求频率**: 1000 请求/分钟
- **并发连接**: 100 个
- **请求大小**: 最大 1MB
- **响应大小**: 最大 10MB

### 资源限制

- **工作空间**: 每个账户最多 10 个
- **应用**: 每个工作空间最多 100 个
- **代理**: 每个工作空间最多 1000 个
- **号码**: 每个工作空间最多 100 个

## 支持和帮助

### 文档资源

- [官方文档](https://docs.fonoster.com)
- [API 参考](https://api.fonoster.com/docs)
- [SDK 文档](https://sdk.fonoster.com)

### 社区支持

- [GitHub Issues](https://github.com/fonoster/fonoster/issues)
- [Discord 社区](https://discord.gg/fonoster)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/fonoster)

### 商业支持

- 邮箱: support@fonoster.com
- 企业支持: enterprise@fonoster.com

---

本文档持续更新，最新版本请访问 [官方文档站点](https://docs.fonoster.com)。