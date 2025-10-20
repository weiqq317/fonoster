# Fonoster 工具模块文档

本文档介绍 Fonoster 项目的主要工具模块，包括 SDK、CLI 工具和 Dashboard 界面。

## 目录

- [SDK 软件开发工具包](#sdk-软件开发工具包)
- [CLI 命令行工具](#cli-命令行工具)
- [Dashboard 管理界面](#dashboard-管理界面)

---

## SDK 软件开发工具包

### 概述

Fonoster SDK 是一个多态软件开发工具包，可在浏览器和 Node.js 环境中使用，为开发者提供与 Fonoster 服务交互的完整接口。

### 核心特性

- **多环境支持**: 支持 Node.js 和浏览器环境
- **完整 API 覆盖**: 涵盖所有 Fonoster 服务的 API 接口
- **多种认证方式**: 支持用户名/密码、API 密钥、刷新令牌认证
- **自动令牌刷新**: SDK 会自动处理令牌过期和刷新
- **TypeScript 支持**: 提供完整的类型定义

### 安装和导入

#### 安装

```bash
# 使用 npm
npm install --save @fonoster/sdk

# 使用 yarn
yarn add @fonoster/sdk

# 浏览器直接引用
<script src="https://unpkg.com/@fonoster/sdk"></script>
```

#### 导入方式

```typescript
// CommonJS 项目
const SDK = require("@fonoster/sdk");

// ES6 模块
import * as SDK from "@fonoster/sdk";

// 浏览器环境
<script src="https://unpkg.com/@fonoster/sdk"></script>
```

### 客户端配置

#### Node.js 环境

```typescript
const SDK = require("@fonoster/sdk");
const ACCESS_KEY_ID = "WO00000000000000000000000000000000";
const ENDPOINT = "api.fonoster.com";
const client = new SDK.Client({ 
  accessKeyId: ACCESS_KEY_ID, 
  endpoint: ENDPOINT 
});
```

#### 浏览器环境

```typescript
const SDK = require("@fonoster/sdk");
const ACCESS_KEY_ID = "WO00000000000000000000000000000000";
const URL = "https://api.fonoster.com/v1beta2";
const client = new SDK.WebClient({ 
  accessKeyId: ACCESS_KEY_ID, 
  url: URL 
});
```

### 认证方式

#### 用户名密码认证

```typescript
const username = "admin@fonoster.local";
const password = "changeme";

await client.login(username, password);
```

#### API 密钥认证

```typescript
const apiKey = "your-api-key";
const apiSecret = "your-api-secret";

await client.loginWithApiKey(apiKey, apiSecret);
```

#### 刷新令牌认证

```typescript
const refreshToken = "your-refresh-token";

await client.loginWithRefreshToken(refreshToken);
```

### 主要服务模块

#### Applications (应用管理)

```typescript
const applications = new SDK.Applications(client);

// 创建应用
const app = await applications.createApplication({
  name: "MyApp",
  type: "EXTERNAL",
  endpoint: "localhost:50061",
  speechToText: {
    productRef: "stt.deepgram",
    config: {
      model: "nova-2",
      languageCode: "en-US"
    }
  },
  textToSpeech: {
    productRef: "tts.elevenlabs",
    config: {
      voice: "lrTWbMInQjSJ9q5ywFKP"
    }
  }
});

// 获取应用列表
const apps = await applications.listApplications({ pageSize: 50 });

// 更新应用
await applications.updateApplication(appRef, updateData);

// 删除应用
await applications.deleteApplication(appRef);
```

#### Calls (通话管理)

```typescript
const calls = new SDK.Calls(client);

// 发起通话
const call = await calls.createCall({
  from: "+18287854037",
  to: "+17853178070",
  appRef: "00000000-0000-0000-0000-000000000000"
});

// 获取通话记录
const callHistory = await calls.listCalls({ pageSize: 100 });
```

#### Agents (SIP 代理)

```typescript
const agents = new SDK.Agents(client);

// 创建 SIP 代理
const agent = await agents.createAgent({
  name: "John Doe",
  username: "1001",
  privacy: "PRIVATE",
  enabled: true,
  maxContacts: 3,
  domainRef: "domain-ref-id"
});
```

#### Domains (域名管理)

```typescript
const domains = new SDK.Domains(client);

// 创建域名
const domain = await domains.createDomain({
  name: "My Domain",
  domainUri: "sip.project.fonoster.io",
  accessControlListRef: "acl-ref-id"
});
```

#### ACLs (访问控制列表)

```typescript
const acls = new SDK.Acls(client);

// 创建 ACL
const acl = await acls.createAcl({
  name: "My ACL",
  allow: ["47.132.130.31", "192.168.1.0/24"]
});
```

#### Trunks (中继线)

```typescript
const trunks = new SDK.Trunks(client);

// 创建中继线
const trunk = await trunks.createTrunk({
  name: "My Trunk",
  inboundUri: "sip.company.fonoster.io"
});
```

#### Numbers (号码管理)

```typescript
const numbers = new SDK.Numbers(client);

// 获取号码列表
const numbers = await numbers.listNumbers({ pageSize: 50 });
```

#### Secrets (密钥管理)

```typescript
const secrets = new SDK.Secrets(client);

// 创建密钥
const secret = await secrets.createSecret({
  name: "api-key",
  secret: "secret-value"
});

// 获取密钥列表
const secretList = await secrets.listSecrets({ pageSize: 50 });
```

#### Users (用户管理)

```typescript
const users = new SDK.Users(client);

// 获取用户信息
const user = await users.getUser();

// 更新用户信息
await users.updateUser(userData);
```

#### Workspaces (工作空间)

```typescript
const workspaces = new SDK.Workspaces(client);

// 获取工作空间列表
const workspaces = await workspaces.listWorkspaces({ pageSize: 50 });

// 创建工作空间
const workspace = await workspaces.createWorkspace({
  name: "My Workspace"
});
```

### 错误处理

```typescript
try {
  const applications = new SDK.Applications(client);
  const response = await applications.createApplication(request);
  console.log(response); // 成功响应
} catch (error) {
  console.error(error); // 错误处理
}
```

### 最佳实践

1. **连接管理**: 重用客户端实例，避免频繁创建连接
2. **错误处理**: 始终使用 try-catch 包装 API 调用
3. **令牌管理**: 利用 SDK 的自动令牌刷新功能
4. **分页处理**: 合理设置 pageSize 参数避免数据过载
5. **环境配置**: 根据部署环境正确配置 endpoint 或 url

---

## CLI 命令行工具

### 概述

Fonoster CLI (ctl) 是基于 oclif 框架构建的命令行工具，用于从命令行管理 Fonoster 资源。支持创建、更新、删除应用、号码、SIP 代理等资源。

### 安装

```bash
npm install -g @fonoster/ctl
```

### 基本使用

```bash
# 查看版本
fonoster --version

# 查看帮助
fonoster --help

# 查看特定命令帮助
fonoster COMMAND --help
```

### 工作空间管理

#### 查看活动工作空间

```bash
fonoster workspaces:active
```

#### 列出所有工作空间

```bash
fonoster workspaces:list
```

#### 切换工作空间

```bash
fonoster workspaces:use <workspace-ref>
```

### API 密钥管理

#### 创建 API 密钥

```bash
fonoster apikeys:create
```

#### 列出 API 密钥

```bash
fonoster apikeys:list
```

#### 重新生成 API 密钥

```bash
fonoster apikeys:regenerate <ref>
```

#### 删除 API 密钥

```bash
fonoster apikeys:delete <ref>
```

### SIP 网络管理

#### ACL (访问控制列表) 管理

```bash
# 列出所有 ACL
fonoster sipnet:acls:list

# 获取特定 ACL
fonoster sipnet:acls:get <ref>

# 创建 ACL
fonoster sipnet:acls:create

# 更新 ACL
fonoster sipnet:acls:update <ref>

# 删除 ACL
fonoster sipnet:acls:delete <ref>
```

#### 域名管理

```bash
# 列出所有域名
fonoster sipnet:domains:list

# 获取特定域名
fonoster sipnet:domains:get <ref>

# 创建域名
fonoster sipnet:domains:create

# 更新域名
fonoster sipnet:domains:update <ref>

# 删除域名
fonoster sipnet:domains:delete <ref>
```

#### 中继线管理

```bash
# 列出所有中继线
fonoster sipnet:trunks:list

# 获取特定中继线
fonoster sipnet:trunks:get <ref>

# 创建中继线
fonoster sipnet:trunks:create

# 更新中继线
fonoster sipnet:trunks:update <ref>

# 删除中继线
fonoster sipnet:trunks:delete <ref>
```

#### 代理管理

```bash
# 列出所有代理
fonoster sipnet:agents:list

# 获取特定代理
fonoster sipnet:agents:get <ref>

# 创建代理
fonoster sipnet:agents:create

# 更新代理
fonoster sipnet:agents:update <ref>

# 删除代理
fonoster sipnet:agents:delete <ref>
```

### 应用管理

```bash
# 列出所有应用
fonoster applications:list

# 获取特定应用
fonoster applications:get <ref>

# 创建应用
fonoster applications:create

# 更新应用
fonoster applications:update <ref>

# 删除应用
fonoster applications:delete <ref>
```

### 号码管理

```bash
# 列出所有号码
fonoster numbers:list

# 获取特定号码
fonoster numbers:get <ref>

# 创建号码
fonoster numbers:create

# 更新号码
fonoster numbers:update <ref>

# 删除号码
fonoster numbers:delete <ref>
```

### 密钥管理

```bash
# 列出所有密钥
fonoster secrets:list

# 获取特定密钥
fonoster secrets:get <ref>

# 创建密钥
fonoster secrets:create

# 更新密钥
fonoster secrets:update <ref>

# 删除密钥
fonoster secrets:delete <ref>
```

### 凭据管理

```bash
# 列出所有凭据
fonoster credentials:list

# 获取特定凭据
fonoster credentials:get <ref>

# 创建凭据
fonoster credentials:create

# 更新凭据
fonoster credentials:update <ref>

# 删除凭据
fonoster credentials:delete <ref>
```

### MCP 配置

```bash
# 配置 MCP 客户端
fonoster mcp:configure --client claude

# 为特定工作空间配置
fonoster mcp:configure --client claude --workspace my-workspace
```

### 全局选项

#### 不安全连接

```bash
# 连接到没有 TLS 的服务器
fonoster COMMAND --insecure
```

#### 分页选项

```bash
# 设置页面大小
fonoster COMMAND --page-size 100
```

### 配置文件

CLI 工具使用配置文件存储工作空间信息和认证凭据，通常位于用户主目录下的 `.fonoster` 文件夹中。

### 最佳实践

1. **安全连接**: 生产环境中避免使用 `--insecure` 选项
2. **工作空间管理**: 使用 `workspaces:use` 切换不同环境
3. **批量操作**: 结合 shell 脚本进行批量资源管理
4. **错误处理**: 检查命令返回状态进行错误处理

---

## Dashboard 管理界面

### 概述

Fonoster Dashboard 是基于 React 和 Material-UI 构建的现代化 Web 管理界面，提供直观的图形化界面来管理 Fonoster 资源。

### 技术栈

- **前端框架**: React 19.0
- **路由**: React Router 7.5
- **UI 组件**: Material-UI 7.0
- **状态管理**: Zustand 5.0
- **数据获取**: TanStack React Query 5.72
- **表单处理**: React Hook Form 7.55
- **类型检查**: TypeScript 5.7
- **构建工具**: Vite 5.4

### 主要功能模块

#### 工作空间概览

- **工作空间信息**: 显示当前工作空间的基本信息
- **快速导航**: 提供到各功能模块的快速访问
- **资源统计**: 显示各类资源的数量统计
- **最近活动**: 展示最近的操作记录

#### 应用管理

- **应用列表**: 查看所有应用及其状态
- **应用创建**: 通过表单创建新应用
- **应用配置**: 编辑应用的各项设置
- **智能评估**: 测试和评估应用的智能功能

#### SIP 网络管理

##### 代理管理
- **代理列表**: 查看所有 SIP 代理
- **代理创建**: 添加新的 SIP 代理
- **代理配置**: 编辑代理的连接设置
- **状态监控**: 实时查看代理连接状态

##### 域名管理
- **域名列表**: 管理 SIP 域名
- **域名配置**: 设置域名的路由规则
- **ACL 关联**: 配置域名的访问控制

##### 中继线管理
- **中继线列表**: 查看所有中继线连接
- **中继线配置**: 设置入站和出站路由
- **连接测试**: 测试中继线连接状态

##### 访问控制列表
- **ACL 管理**: 创建和管理 IP 访问控制
- **规则配置**: 设置允许和拒绝的 IP 范围
- **安全策略**: 配置网络安全策略

#### 号码管理

- **号码列表**: 查看所有可用号码
- **号码分配**: 将号码分配给应用
- **路由配置**: 设置号码的路由规则
- **使用统计**: 查看号码的使用情况

#### 凭据管理

- **凭据列表**: 管理各种认证凭据
- **凭据创建**: 添加新的认证凭据
- **安全配置**: 设置凭据的安全策略
- **权限管理**: 配置凭据的访问权限

#### 密钥管理

- **密钥列表**: 管理应用密钥和配置
- **密钥创建**: 添加新的配置密钥
- **值编辑**: 安全地编辑密钥值
- **使用跟踪**: 跟踪密钥的使用情况

#### API 密钥管理

- **密钥列表**: 查看所有 API 密钥
- **密钥生成**: 创建新的 API 密钥
- **权限配置**: 设置 API 密钥的权限范围
- **使用监控**: 监控 API 密钥的使用情况

#### 工作空间设置

- **基本信息**: 编辑工作空间的基本信息
- **成员管理**: 邀请和管理工作空间成员
- **权限配置**: 设置成员的角色和权限
- **安全设置**: 配置工作空间的安全策略

### 用户界面特性

#### 响应式设计

- **移动适配**: 支持移动设备访问
- **自适应布局**: 根据屏幕尺寸调整界面
- **触摸友好**: 优化触摸操作体验

#### 数据表格

- **排序功能**: 支持多列排序
- **筛选功能**: 提供高级筛选选项
- **分页显示**: 支持大数据集分页
- **导出功能**: 支持数据导出为 CSV

#### 表单处理

- **实时验证**: 表单字段实时验证
- **错误提示**: 清晰的错误信息显示
- **自动保存**: 支持表单自动保存
- **批量操作**: 支持批量编辑和删除

#### 通知系统

- **操作反馈**: 实时操作结果通知
- **错误提示**: 详细的错误信息显示
- **成功确认**: 操作成功的确认提示
- **进度指示**: 长时间操作的进度显示

### 开发和部署

#### 开发环境

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 类型检查
npm run typecheck

# 构建 Storybook
npm run build:storybook
```

#### 生产部署

```bash
# 构建生产版本
npm run build

# 启动生产服务器
npm start
```

#### 环境配置

Dashboard 通过环境变量进行配置：

- `PORT`: 服务器端口 (默认: 3030)
- `API_ENDPOINT`: Fonoster API 服务器地址
- `AUTH_DOMAIN`: 认证服务域名

### 安全特性

#### 认证和授权

- **JWT 令牌**: 使用 JWT 进行用户认证
- **自动刷新**: 令牌自动刷新机制
- **权限控制**: 基于角色的访问控制
- **会话管理**: 安全的会话管理

#### 数据保护

- **HTTPS 强制**: 强制使用 HTTPS 连接
- **数据加密**: 敏感数据传输加密
- **XSS 防护**: 跨站脚本攻击防护
- **CSRF 防护**: 跨站请求伪造防护

### 最佳实践

1. **性能优化**: 使用 React Query 进行数据缓存
2. **用户体验**: 提供加载状态和错误处理
3. **可访问性**: 遵循 WCAG 可访问性标准
4. **代码质量**: 使用 TypeScript 和 ESLint
5. **测试覆盖**: 编写单元测试和集成测试

### 故障排除

#### 常见问题

1. **连接问题**: 检查 API 服务器地址配置
2. **认证失败**: 验证用户凭据和令牌有效性
3. **权限错误**: 确认用户角色和权限设置
4. **性能问题**: 检查网络连接和服务器响应时间

#### 调试工具

- **React DevTools**: React 组件调试
- **Redux DevTools**: 状态管理调试
- **Network Tab**: 网络请求监控
- **Console Logs**: 应用日志查看

---

## 总结

Fonoster 的工具模块为开发者和管理员提供了完整的工具链：

- **SDK**: 为开发者提供编程接口，支持多种语言和环境
- **CLI**: 为系统管理员提供命令行工具，适合自动化和脚本操作
- **Dashboard**: 为最终用户提供图形化界面，便于日常管理操作

这三个工具相互补充，覆盖了从开发、部署到运维的完整生命周期，为 Fonoster 平台的使用提供了灵活多样的选择。