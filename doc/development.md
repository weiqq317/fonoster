# Fonoster 开发环境搭建和贡献指南

本文档提供了 Fonoster 项目的完整开发环境搭建指南和贡献流程说明。

## 目录

- [环境要求](#环境要求)
- [快速开始](#快速开始)
- [开发环境配置](#开发环境配置)
- [项目结构](#项目结构)
- [开发工作流](#开发工作流)
- [代码规范](#代码规范)
- [测试指南](#测试指南)
- [构建和部署](#构建和部署)
- [贡献流程](#贡献流程)
- [故障排除](#故障排除)

## 环境要求

### 系统要求
- **操作系统**: Linux, macOS, Windows (推荐 Linux/macOS)
- **Node.js**: 20.11.1 或更高版本
- **npm**: 最新版本
- **Docker**: 20.10+ 和 Docker Compose v2
- **Git**: 2.30+

### 推荐工具
- **IDE**: Visual Studio Code 或 WebStorm
- **终端**: 支持 ANSI 颜色的现代终端
- **数据库客户端**: DBeaver, pgAdmin 或 Adminer

### 硬件要求
- **内存**: 最少 8GB RAM (推荐 16GB+)
- **存储**: 至少 10GB 可用空间
- **网络**: 稳定的互联网连接

## 快速开始

### 1. 克隆项目

```bash
# 克隆主仓库
git clone https://github.com/fonoster/fonoster.git
cd fonoster

# 或者 fork 后克隆你的分支
git clone https://github.com/YOUR_USERNAME/fonoster.git
cd fonoster
```

### 2. 安装依赖

```bash
# 安装所有依赖 (使用 Lerna monorepo 管理)
npm install

# 验证安装
npm run build
```

### 3. 环境配置

```bash
# 复制环境变量模板
cp .env.example .env

# 生成 RSA 密钥对
npm run generate:keypair

# 或者手动生成
mkdir -p .keys
openssl genrsa -out .keys/private.pem 2048
openssl rsa -in .keys/private.pem -pubout -out .keys/public.pem
```

### 4. 启动开发环境

```bash
# 启动基础服务 (PostgreSQL, InfluxDB, NATS, Routr, Asterisk 等)
npm run start:services

# 等待服务启动完成 (约 30-60 秒)
docker compose ps

# 启动 API 服务器
npm run start:apiserver

# 启动 Dashboard (新终端)
npm run start:dashboard

# 启动语音服务 (可选)
npm run start:voice

# 启动 AI 助手服务 (可选)
npm run start:autopilot
```

### 5. 验证安装

访问以下地址验证服务是否正常运行：

- **Dashboard**: http://localhost:3030
- **API Server**: http://localhost:50051 (gRPC)
- **Adminer** (数据库管理): http://localhost:8282
- **MailHog** (邮件测试): http://localhost:8025

## 开发环境配置

### 环境变量说明

编辑 `.env` 文件，配置以下关键变量：

```bash
# 必须配置的 IP 地址 (本地开发使用本机 IP)
ROUTR_EXTERNAL_ADDRS=192.168.1.100  # 替换为你的本机 IP
ASTERISK_SIPPROXY_HOST=192.168.1.100
RTPENGINE_PUBLIC_IP=192.168.1.100

# 管理员账户
APISERVER_OWNER_EMAIL=admin@fonoster.local
APISERVER_OWNER_PASSWORD=changeme

# 数据库配置
POSTGRES_PASSWORD=postgres
APISERVER_DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fonoster
APISERVER_IDENTITY_DATABASE_URL=postgresql://postgres:postgres@postgres:5432/fnidentity

# 开发模式
NODE_ENV=development
```

### IDE 配置

#### Visual Studio Code

推荐安装以下扩展：

```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "esbenp.prettier-vscode",
    "ms-vscode.eslint",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-json",
    "ms-vscode.docker",
    "ms-vscode.remote-containers"
  ]
}
```

工作区设置 (`.vscode/settings.json`):

```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "eslint.workingDirectories": ["mods/*"],
  "typescript.preferences.includePackageJsonAutoImports": "on"
}
```

## 项目结构

```
fonoster/
├── mods/                    # 核心模块 (Lerna monorepo)
│   ├── apiserver/          # API 服务器
│   ├── dashboard/          # Web 管理界面
│   ├── sdk/               # 开发工具包
│   ├── ctl/               # 命令行工具
│   ├── identity/          # 身份认证服务
│   ├── voice/             # 语音服务
│   ├── autopilot/         # AI 助手
│   ├── sipnet/            # SIP 网络栈
│   ├── common/            # 公共库
│   ├── types/             # 类型定义
│   ├── logger/            # 日志服务
│   ├── streams/           # 音频流处理
│   ├── authz/             # 授权服务
│   └── mcp/               # 模型上下文协议
├── config/                 # 配置文件
│   ├── keys/              # RSA 密钥对
│   ├── envoy.yaml         # Envoy 代理配置
│   └── integrations.json  # 集成配置
├── asterisk/              # Asterisk 配置
├── doc/                   # 文档
├── .scripts/              # 构建脚本
├── compose.yaml           # 生产环境 Docker Compose
├── compose.dev.yaml       # 开发环境 Docker Compose
├── package.json           # 根项目配置
├── tsconfig.json          # TypeScript 配置
├── eslint.config.mjs      # ESLint 配置
└── .env.example           # 环境变量模板
```

### 模块依赖关系

```mermaid
graph TD
    A[API Server] --> B[Identity]
    A --> C[Voice]
    A --> D[Autopilot]
    A --> E[SIPNet]
    A --> F[Common]
    A --> G[Types]
    A --> H[Logger]
    
    I[Dashboard] --> J[SDK]
    K[CLI] --> J
    
    J --> F
    J --> G
    
    B --> F
    C --> F
    D --> F
    E --> F
    
    L[Authorization] --> B
    M[Streams] --> H
    N[MCP] --> F
```

## 开发工作流

### 1. 分支管理

```bash
# 创建功能分支
git checkout -b feature/your-feature-name

# 创建修复分支
git checkout -b fix/issue-description

# 创建文档分支
git checkout -b docs/documentation-update
```

### 2. 开发流程

```bash
# 1. 安装依赖
npm install

# 2. 启动开发服务
npm run start:services
npm run start:apiserver

# 3. 进行开发
# 编辑代码...

# 4. 运行测试
npm test

# 5. 检查代码质量
npm run lint
npm run lint:fix

# 6. 构建项目
npm run build
```

### 3. 模块开发

#### 创建新模块

```bash
# 1. 在 mods/ 目录下创建新模块
mkdir mods/your-module
cd mods/your-module

# 2. 初始化 package.json
npm init -y

# 3. 创建基本结构
mkdir -p src test
touch src/index.ts
touch tsconfig.json
touch README.md

# 4. 配置 TypeScript
cat > tsconfig.json << EOF
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "references": [
    {
      "path": "../common"
    }
  ],
  "exclude": ["node_modules", "dist", "test"]
}
EOF

# 5. 更新根目录 package.json 的 workspaces
```

#### 模块间通信

```typescript
// 使用 gRPC 进行模块间通信
import { createChannel, createClient } from '@grpc/grpc-js';
import { IdentityClient } from '@fonoster/types';

const channel = createChannel('localhost:50051');
const client = createClient(IdentityClient, channel);
```

### 4. 数据库开发

#### Prisma 操作

```bash
# 生成 Prisma 客户端
npm run generate:prisma-clients

# 创建迁移
cd mods/apiserver
npx prisma migrate dev --name your-migration-name

# 重置数据库
npx prisma migrate reset

# 查看数据库
npx prisma studio
```

#### 数据库模式更新

```prisma
// mods/apiserver/schema.prisma
model YourModel {
  id        String   @id @default(cuid())
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@map("your_models")
}
```

## 代码规范

### TypeScript 规范

```typescript
// 1. 使用严格的类型定义
interface UserRequest {
  name: string;
  email: string;
  age?: number;
}

// 2. 使用枚举而不是字符串常量
enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  GUEST = 'guest'
}

// 3. 使用泛型提高代码复用性
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

// 4. 正确的错误处理
async function fetchUser(id: string): Promise<User> {
  try {
    const user = await userService.findById(id);
    if (!user) {
      throw new Error(`User with id ${id} not found`);
    }
    return user;
  } catch (error) {
    logger.error('Failed to fetch user', { id, error });
    throw error;
  }
}
```

### 文件头注释

所有 TypeScript 文件必须包含版权声明：

```typescript
/**
 * Copyright (C) 2025 by Fonoster Inc (https://fonoster.com)
 * http://github.com/fonoster/fonoster
 *
 * This file is part of Fonoster
 *
 * Licensed under the MIT License (the "License");
 * you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 *    https://opensource.org/licenses/MIT
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
```

### 命名规范

```typescript
// 文件名: camelCase
// userService.ts, apiClient.ts

// 类名: PascalCase
class UserService {}
class ApiClient {}

// 接口名: PascalCase (可选 I 前缀)
interface User {}
interface IUserRepository {}

// 变量和函数: camelCase
const userName = 'john';
function getUserById() {}

// 常量: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.fonoster.com';

// 枚举: PascalCase
enum UserStatus {
  ACTIVE = 'active',
  INACTIVE = 'inactive'
}
```

### ESLint 配置

项目使用严格的 ESLint 规则：

```bash
# 检查代码质量
npm run lint

# 自动修复可修复的问题
npm run lint:fix
```

主要规则：
- 禁止使用 `any` 类型 (警告)
- 强制使用 Prettier 格式化
- 禁止未使用的变量
- 强制导入排序
- 禁止使用循环 (推荐函数式编程)

## 测试指南

### 测试结构

```
mods/your-module/
├── src/
│   └── index.ts
└── test/
    ├── unit/
    │   └── index.test.ts
    ├── integration/
    │   └── api.integration.ts
    └── fixtures/
        └── testData.json
```

### 单元测试

```typescript
// test/unit/userService.test.ts
import { expect } from 'chai';
import { UserService } from '../src/userService';

describe('UserService', () => {
  let userService: UserService;

  beforeEach(() => {
    userService = new UserService();
  });

  describe('createUser', () => {
    it('should create a user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com'
      };

      const user = await userService.createUser(userData);

      expect(user).to.have.property('id');
      expect(user.name).to.equal(userData.name);
      expect(user.email).to.equal(userData.email);
    });

    it('should throw error for invalid email', async () => {
      const userData = {
        name: 'John Doe',
        email: 'invalid-email'
      };

      await expect(userService.createUser(userData))
        .to.be.rejectedWith('Invalid email format');
    });
  });
});
```

### 集成测试

```typescript
// test/integration/api.integration.ts
import { expect } from 'chai';
import { SDK } from '@fonoster/sdk';

describe('API Integration', () => {
  let client: SDK.Client;

  before(async () => {
    client = new SDK.Client();
    await client.loginWithApiKey('test-api-key');
  });

  after(async () => {
    await client.close();
  });

  it('should create and retrieve user', async () => {
    const userData = {
      name: 'Test User',
      email: 'test@example.com'
    };

    const createdUser = await client.users.create(userData);
    expect(createdUser).to.have.property('id');

    const retrievedUser = await client.users.get(createdUser.id);
    expect(retrievedUser.name).to.equal(userData.name);
  });
});
```

### 运行测试

```bash
# 运行所有单元测试
npm test

# 运行集成测试
npm run integration

# 运行 Web 集成测试
npm run integration:web

# 运行特定模块的测试
npm test -- --grep "UserService"

# 生成测试覆盖率报告
npm run test:coverage
```

### 测试环境配置

测试使用独立的环境配置：

```bash
# 测试前置条件检查
npm run pretest

# 设置测试环境变量
export NODE_ENV=test
export LOGS_LEVEL=none
```

## 构建和部署

### 本地构建

```bash
# 构建所有模块
npm run build

# 清理构建产物
npm run clean

# 重新构建
npm run clean && npm run build
```

### Docker 构建

```bash
# 构建开发环境
docker compose -f compose.yaml -f compose.dev.yaml build

# 构建生产环境
docker compose build

# 构建特定服务
docker compose build apiserver
```

### 发布流程

```bash
# 检查更新
npm run check:updates

# 版本管理 (使用 Lerna)
npm run bump:version

# 发布到 npm
npm run release

# 发布预发布版本
npm run release:prerelease
```

## 贡献流程

### 1. 准备工作

```bash
# Fork 项目到你的 GitHub 账户
# 克隆你的 fork
git clone https://github.com/YOUR_USERNAME/fonoster.git
cd fonoster

# 添加上游仓库
git remote add upstream https://github.com/fonoster/fonoster.git

# 获取最新代码
git fetch upstream
git checkout main
git merge upstream/main
```

### 2. 开发流程

```bash
# 创建功能分支
git checkout -b feature/your-feature-name

# 进行开发
# ... 编写代码 ...

# 提交代码 (使用 Conventional Commits)
git add .
git commit -m "feat: add user authentication feature"

# 推送到你的 fork
git push origin feature/your-feature-name
```

### 3. 提交规范

使用 [Conventional Commits](https://www.conventionalcommits.org/) 规范：

```bash
# 功能
git commit -m "feat: add user registration API"

# 修复
git commit -m "fix: resolve authentication token expiration issue"

# 文档
git commit -m "docs: update API documentation"

# 样式
git commit -m "style: format code with prettier"

# 重构
git commit -m "refactor: extract user validation logic"

# 测试
git commit -m "test: add unit tests for user service"

# 构建
git commit -m "build: update dependencies"

# CI/CD
git commit -m "ci: add GitHub Actions workflow"

# 其他
git commit -m "chore: update package.json"
```

### 4. Pull Request

1. **创建 PR**: 在 GitHub 上创建 Pull Request
2. **填写模板**: 使用 PR 模板填写详细信息
3. **代码审查**: 等待维护者审查
4. **修改反馈**: 根据反馈修改代码
5. **合并**: 审查通过后合并到主分支

#### PR 检查清单

- [ ] 代码遵循项目规范
- [ ] 添加了必要的测试
- [ ] 测试全部通过
- [ ] 文档已更新
- [ ] 提交信息符合规范
- [ ] 没有合并冲突

### 5. 代码审查

#### 审查要点

1. **功能正确性**: 代码是否实现了预期功能
2. **代码质量**: 是否遵循最佳实践
3. **性能**: 是否有性能问题
4. **安全性**: 是否存在安全漏洞
5. **测试覆盖**: 是否有足够的测试
6. **文档**: 是否更新了相关文档

#### 审查流程

```bash
# 检出 PR 分支进行本地测试
git fetch origin pull/123/head:pr-123
git checkout pr-123

# 运行测试
npm test
npm run lint

# 本地验证功能
npm run start:services
npm run start:apiserver
```

## 故障排除

### 常见问题

#### 1. 依赖安装失败

```bash
# 清理缓存
npm cache clean --force
rm -rf node_modules package-lock.json

# 重新安装
npm install
```

#### 2. Docker 服务启动失败

```bash
# 检查服务状态
docker compose ps

# 查看日志
docker compose logs -f [service-name]

# 重启服务
docker compose restart [service-name]

# 完全重建
docker compose down
docker compose up -d --build
```

#### 3. 数据库连接问题

```bash
# 检查数据库状态
docker compose exec postgres psql -U postgres -l

# 重置数据库
npm run db:reset

# 手动迁移
cd mods/apiserver
npx prisma migrate deploy
```

#### 4. 端口冲突

```bash
# 检查端口占用
lsof -i :3030
lsof -i :50051

# 修改端口配置
# 编辑 .env 文件或 docker-compose 配置
```

#### 5. TypeScript 编译错误

```bash
# 清理 TypeScript 缓存
find . -name "tsconfig.tsbuildinfo" -delete
find . -name "dist" -type d -exec rm -rf {} +

# 重新构建
npm run build
```

#### 6. gRPC 连接问题

```bash
# 检查 gRPC 服务
grpcurl -plaintext localhost:50051 list

# 测试连接
npm run test:grpc
```

### 调试技巧

#### 1. 启用详细日志

```bash
# 设置日志级别
export LOGS_LEVEL=verbose
export NODE_ENV=development

# 启动服务
npm run start:apiserver
```

#### 2. 使用调试器

```typescript
// 在代码中添加断点
debugger;

// 或使用 console.log
console.log('Debug info:', { variable });
```

#### 3. 数据库调试

```bash
# 使用 Prisma Studio
cd mods/apiserver
npx prisma studio

# 或使用 Adminer
# 访问 http://localhost:8282
```

#### 4. 网络调试

```bash
# 检查容器网络
docker network ls
docker network inspect fonoster_default

# 测试容器间连接
docker compose exec apiserver ping postgres
```

### 性能优化

#### 1. 开发环境优化

```bash
# 使用 nodemon 进行热重载
npm run start:apiserver  # 已配置 nodemon

# 并行启动服务
npm run start:services & npm run start:apiserver
```

#### 2. 构建优化

```bash
# 并行构建
npm run build --parallel

# 增量构建
npm run build --since HEAD~1
```

#### 3. 测试优化

```bash
# 并行测试
npm test --parallel

# 只运行变更的测试
npm test --since HEAD~1
```

## 社区和支持

### 获取帮助

- **Discord**: [加入 Fonoster Discord 社区](https://discord.gg/4QWgSz4hTC)
- **GitHub Issues**: [报告问题或请求功能](https://github.com/fonoster/fonoster/issues)
- **GitHub Discussions**: [参与讨论](https://github.com/fonoster/fonoster/discussions)
- **文档**: [查看完整文档](https://docs.fonoster.com)

### 贡献方式

- 🐛 **报告 Bug**: 发现问题请创建 Issue
- 💡 **功能建议**: 提出新功能想法
- 📝 **改进文档**: 帮助完善文档
- 🧪 **编写测试**: 提高测试覆盖率
- 💻 **贡献代码**: 修复 Bug 或实现新功能
- 🎨 **UI/UX 改进**: 优化用户界面和体验
- 🌐 **国际化**: 帮助翻译项目

### 行为准则

请遵循我们的 [行为准则](https://github.com/fonoster/fonoster/blob/main/CODE_OF_CONDUCT.md)，创建一个友好和包容的社区环境。

---

## 许可证

本项目采用 MIT 许可证。详情请参阅 [LICENSE](../LICENSE) 文件。

---

**感谢你对 Fonoster 项目的贡献！** 🎉

如果你在开发过程中遇到任何问题，请不要犹豫在 Discord 或 GitHub 上寻求帮助。我们的社区非常乐意帮助新贡献者。