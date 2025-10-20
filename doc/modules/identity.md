# Identity 模块文档

## 概述

Identity 模块是 Fonoster 生态系统中安全用户管理、身份认证和授权的基石。它提供了灵活且可扩展的身份管理解决方案，能够适应 Fonoster 各个项目的多样化和不断发展的需求。

## 核心功能

### 1. 用户管理
- 用户账户创建、读取、更新和删除 (CRUD)
- 个人用户账户和服务账户支持
- 用户资料管理（头像、联系信息等）
- 邮箱和电话号码验证

### 2. 工作空间管理
- 多租户工作空间支持
- 工作空间成员管理
- 基于角色的权限控制
- 工作空间邀请和加入流程

### 3. 身份认证
- 用户名密码认证
- 多因素认证 (MFA) 支持
- OAuth2 集成（GitHub 等第三方身份提供商）
- API 密钥认证
- JWT 令牌管理

### 4. 授权控制
- 基于角色的访问控制 (RBAC)
- 细粒度权限管理
- 预定义角色和自定义角色
- 方法级访问控制

## 架构设计

### 目录结构

```
mods/identity/
├── src/
│   ├── apikeys/            # API 密钥管理
│   ├── exchanges/          # 令牌交换服务
│   ├── invites/            # 邀请管理
│   ├── users/              # 用户管理
│   ├── workspaces/         # 工作空间管理
│   ├── utils/              # 工具函数
│   ├── db/                 # 数据库配置
│   ├── service.ts          # 服务构建器
│   └── index.ts            # 模块导出
├── migrations/             # 数据库迁移文件
├── schema.prisma           # Prisma 数据模型
├── package.json
└── README.md
```

### 核心组件

#### 1. 服务构建器 (`service.ts`)

负责构建和配置 Identity 服务：

```typescript
function buildIdentityService(identityConfig: IdentityConfig) {
  const prisma = createPrismaClient(
    identityConfig.dbUrl,
    identityConfig.encryptionKey
  );

  return {
    // 用户管理
    createUser: createCreateUser(prisma, identityConfig),
    getUser: createGetUser(prisma),
    updateUser: createUpdateUser(prisma),
    deleteUser: createDeleteUser(prisma),
    
    // 工作空间管理
    createWorkspace: createCreateWorkspace(prisma),
    getWorkspace: createGetWorkspace(prisma),
    updateWorkspace: createUpdateWorkspace(prisma),
    listWorkspaces: createListWorkspaces(prisma),
    
    // 令牌交换
    exchangeApiKey: createExchangeApiKey(prisma, identityConfig),
    exchangeCredentials: createExchangeCredentials(prisma, identityConfig),
    exchangeOauth2Code: createExchangeOauth2Code(prisma, identityConfig),
    exchangeRefreshToken: createExchangeRefreshToken(prisma, identityConfig),
    
    // 公钥获取
    getPublicKey: createGetPublicKey(identityConfig.publicKey),
    
    // API 密钥管理
    createApiKey: createCreateApiKey(prisma),
    regenerateApiKey: createRegenerateApiKey(prisma),
    deleteApiKey: createDeleteApiKey(prisma),
    listApiKeys: createListApiKeys(prisma)
  };
}
```

#### 2. 配置类型 (`exchanges/types.ts`)

定义 Identity 服务的配置结构：

```typescript
type IdentityConfig = {
  dbUrl: string;                              // 数据库连接 URL
  issuer: string;                             // JWT 发行者
  audience: string;                           // JWT 受众
  privateKey: string;                         // JWT 签名私钥
  publicKey: string;                          // JWT 验证公钥
  encryptionKey: string;                      // 数据加密密钥
  accessTokenExpiresIn: number | string;      // 访问令牌过期时间
  refreshTokenExpiresIn: number | string;     // 刷新令牌过期时间
  idTokenExpiresIn: number | string;          // ID 令牌过期时间
  workspaceInviteExpiration: string;          // 工作空间邀请过期时间
  workspaceInviteUrl: string;                 // 工作空间邀请 URL
  workspaceInviteFailUrl: string;             // 邀请失败重定向 URL
  contactVerificationRequired: boolean;       // 是否需要联系方式验证
  twoFactorAuthenticationRequired: boolean;   // 是否需要双因素认证
  smtpConfig: {                              // SMTP 邮件配置
    sender: string;
    host: string;
    port: number;
    secure: boolean;
    auth: {
      user: string;
      pass: string;
    };
  };
  twilioSmsConfig?: {                        // Twilio 短信配置
    accountSid: string;
    authToken: string;
    sender: string;
  };
  githubOauth2Config?: {                     // GitHub OAuth2 配置
    clientId: string;
    clientSecret: string;
  };
};
```

## 数据模型

### Prisma Schema 主要模型

#### 1. 用户模型 (User)

```prisma
model User {
  ref                 String   @id @default(uuid())
  accessKeyId         String   @unique @map("access_key_id") @db.VarChar(255)
  name                String   @db.VarChar(60)
  email               String   @unique @db.VarChar(255)
  emailVerified       Boolean  @default(false) @map("email_verified")
  password            String   @map("password_hash") /// @encrypted
  phoneNumber         String?  @map("phone_number") @db.VarChar(20)
  phoneNumberVerified Boolean  @default(false) @map("phone_number_verified")
  avatar              String?  @db.VarChar(255)
  createdAt           DateTime @default(now()) @map("created_at") @db.Timestamptz(3)
  updatedAt           DateTime @default(now()) @map("updated_at") @db.Timestamptz(3)
  extended            Json?

  // 关系
  ownedWorkspaces Workspace[]       // 用户拥有的工作空间
  memberships     WorkspaceMember[] // 用户参与的工作空间

  @@index([email], type: Hash)
  @@index([accessKeyId], type: Hash)
  @@map("users")
}
```

#### 2. 工作空间模型 (Workspace)

```prisma
model Workspace {
  ref         String   @id @default(uuid())
  accessKeyId String   @unique @map("access_key_id") @db.VarChar(255)
  name        String   @db.VarChar(60)
  createdAt   DateTime @default(now()) @map("created_at") @db.Timestamptz(3)
  updatedAt   DateTime @default(now()) @map("updated_at") @db.Timestamptz(3)

  // 关系
  owner    User              @relation(fields: [ownerRef], references: [ref], onDelete: Cascade)
  ownerRef String            @map("owner_ref")
  members  WorkspaceMember[]
  apiKeys  ApiKey[]

  @@index([accessKeyId], type: Hash)
  @@index([ownerRef], type: Hash)
  @@map("workspaces")
}
```

#### 3. 工作空间成员模型 (WorkspaceMember)

```prisma
model WorkspaceMember {
  ref       String                @id @default(uuid())
  status    WorkspaceMemberStatus @default(PENDING)
  role      Role                  @default(WORKSPACE_MEMBER)
  createdAt DateTime              @default(now()) @map("created_at") @db.Timestamptz(3)
  updatedAt DateTime              @default(now()) @map("updated_at") @db.Timestamptz(3)

  // 关系
  user         User      @relation(fields: [userRef], references: [ref], onDelete: Cascade)
  userRef      String    @map("user_ref")
  workspace    Workspace @relation(fields: [workspaceRef], references: [ref], onDelete: Cascade)
  workspaceRef String    @map("workspace_ref")

  @@unique([userRef, workspaceRef])
  @@map("workspace_members")
}
```

#### 4. API 密钥模型 (ApiKey)

```prisma
model ApiKey {
  ref             String    @id @default(uuid())
  accessKeyId     String    @unique @map("access_key_id") @db.VarChar(255)
  accessKeySecret String    @map("access_key_secret") @db.VarChar(255) /// @encrypted
  role            Role      @default(WORKSPACE_MEMBER)
  createdAt       DateTime  @default(now()) @map("created_at") @db.Timestamptz(3)
  updatedAt       DateTime  @default(now()) @map("updated_at") @db.Timestamptz(3)
  expiresAt       DateTime? @map("expires_at") @db.Timestamptz(3)

  // 关系
  workspace    Workspace @relation(fields: [workspaceRef], references: [ref], onDelete: Cascade)
  workspaceRef String    @map("workspace_ref")

  @@index([accessKeyId], type: Hash)
  @@index([workspaceRef], type: Hash)
  @@map("api_keys")
}
```

#### 5. 验证码模型 (VerificationCode)

```prisma
model VerificationCode {
  ref       String           @id @default(uuid())
  type      VerificationType
  code      String           @db.VarChar(6)
  value     String           @db.VarChar(255)
  expiresAt DateTime         @map("expires_at") @db.Timestamptz(3)
  createdAt DateTime         @default(now()) @map("created_at") @db.Timestamptz(3)

  @@index([code], type: Hash)
  @@map("verification_codes")
}
```

### 枚举类型

```prisma
enum VerificationType {
  EMAIL
  PHONE
}

enum WorkspaceMemberStatus {
  PENDING
  ACTIVE
}

enum Role {
  USER
  WORKSPACE_ADMIN
  WORKSPACE_OWNER
  WORKSPACE_MEMBER
}
```

## JWT 令牌系统

### 令牌类型

Identity 模块使用三种类型的 JWT 令牌：

#### 1. ID 令牌 (ID Token)

用于标识用户身份，包含用户基本信息：

```json
{
  "iss": "https://identity-global.fonoster.com",
  "sub": "00000000-0000-0000-0000-000000000000",
  "aud": "api",
  "tokenUse": "id",
  "accessKeyId": "US00000000000000000000000000000000",
  "email": "johndoe@example.com",
  "emailVerified": false,
  "phoneNumber": null,
  "phoneNumberVerified": false,
  "iat": 1723477780,
  "exp": 1723478680
}
```

#### 2. 访问令牌 (Access Token)

用于 API 访问授权，包含权限信息：

```json
{
  "iss": "https://identity-global.fonoster.com",
  "sub": "00000000-0000-0000-0000-000000000000",
  "aud": "api",
  "tokenUse": "access",
  "accessKeyId": "US00000000000000000000000000000000",
  "access": [
    {
      "accessKeyId": "US00000000000000000000000000000000",
      "role": "user"
    }
  ],
  "iat": 1723477780,
  "exp": 1723481380
}
```

#### 3. 刷新令牌 (Refresh Token)

用于获取新的访问令牌：

```json
{
  "iss": "https://identity-global.fonoster.com",
  "sub": "00000000-0000-0000-0000-000000000000",
  "aud": "api",
  "tokenUse": "refresh",
  "accessKeyId": "US00000000000000000000000000000000",
  "iat": 1723477780,
  "exp": 1731253780
}
```

### 令牌生成和验证

#### 令牌生成

```typescript
function createGenerateCallAccessToken(identityConfig: IdentityConfig) {
  return async function generateCallAccessToken(params: {
    accessKeyId: string;
    appRef: string;
  }): Promise<string> {
    const { privateKey } = identityConfig;
    const { issuer, audience } = identityConfig;
    const { accessKeyId, appRef } = params;

    const accessTokenSignOptions = {
      algorithm: SIGN_ALGORITHM,
      expiresIn: "30s"  // 短期有效期
    } as jwt.SignOptions;

    const access = [
      {
        accessKeyId,
        role: VOICE_SERVICE_ROLE
      }
    ];

    const unsignedToken = {
      iss: issuer,
      sub: appRef,
      aud: audience,
      tokenUse: TokenUseEnum.ACCESS,
      accessKeyId,
      access
    };

    return jwt.sign(unsignedToken, privateKey, accessTokenSignOptions);
  };
}
```

#### 令牌验证

使用 RS256 算法验证令牌：

1. 使用公钥验证令牌签名
2. 验证令牌声明（发行者、受众、过期时间等）

```typescript
// 获取公钥进行验证
const publicKey = await identityService.getPublicKey();
const isValid = jwt.verify(token, publicKey, { algorithms: ['RS256'] });
```

## 认证机制

### 1. 用户名密码认证

```typescript
const exchangeCredentials = createExchangeCredentials(prisma, identityConfig);

const tokens = await exchangeCredentials({
  username: "user@example.com",
  password: "securePassword"
});

// 返回 { idToken, accessToken, refreshToken }
```

### 2. OAuth2 认证

支持 GitHub OAuth2 集成：

```typescript
const exchangeOauth2Code = createExchangeOauth2Code(prisma, identityConfig);

const tokens = await exchangeOauth2Code({
  code: "github_authorization_code",
  provider: "github"
});
```

#### GitHub OAuth2 流程

```typescript
async function getGitHubUserWithOauth2Code(params: {
  clientId: string;
  clientSecret: string;
  code: string;
}) {
  // 1. 使用授权码获取访问令牌
  const tokenResponse = await fetch(GITHUB_API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Accept: "application/json"
    },
    body: JSON.stringify({
      client_id: clientId,
      client_secret: clientSecret,
      code
    })
  });

  const tokenData = await tokenResponse.json();
  const accessToken = tokenData?.access_token;

  // 2. 使用访问令牌获取用户信息
  const userResponse = await fetch(GITHUB_USER_API_URL, {
    headers: {
      Authorization: `token ${accessToken}`
    }
  });

  return userResponse.json();
}
```

### 3. API 密钥认证

```typescript
const exchangeApiKey = createExchangeApiKey(prisma, identityConfig);

const tokens = await exchangeApiKey({
  accessKeyId: "US00000000000000000000000000000000",
  accessKeySecret: "secret_key_value"
});
```

### 4. 刷新令牌认证

```typescript
const exchangeRefreshToken = createExchangeRefreshToken(prisma, identityConfig);

const tokens = await exchangeRefreshToken({
  refreshToken: "existing_refresh_token"
});
```

## 角色和权限

### 预定义角色

```typescript
enum Role {
  USER,                 // 普通用户
  WORKSPACE_ADMIN,      // 工作空间管理员
  WORKSPACE_OWNER,      // 工作空间所有者
  WORKSPACE_MEMBER      // 工作空间成员
}
```

### 角色权限示例

```json
{
  "name": "user",
  "description": "Access to User and Workspace endpoints",
  "access": [
    "/fonoster.identity.v1beta2.Identity/GetUser",
    "/fonoster.identity.v1beta2.Identity/UpdateUser",
    "/fonoster.identity.v1beta2.Identity/DeleteUser",
    "/fonoster.identity.v1beta2.Identity/CreateWorkspace",
    "/fonoster.identity.v1beta2.Identity/GetWorkspace",
    "/fonoster.identity.v1beta2.Identity/UpdateWorkspace",
    "/fonoster.identity.v1beta2.Identity/ListWorkspaces",
    "/fonoster.identity.v1beta2.Identity/RefreshToken"
  ]
}
```

## 工作空间管理

### 工作空间邀请流程

#### 1. 生成邀请令牌

```typescript
function createGenerateWorkspaceInviteToken(identityConfig: IdentityConfig) {
  return async function generateWorkspaceInviteToken(params: {
    userRef: string;
    memberRef: string;
    accessKeyId: string;
    expiresIn?: string;
  }): Promise<string> {
    const { privateKey } = identityConfig;
    const { issuer, audience } = identityConfig;
    const { userRef, memberRef, accessKeyId, expiresIn } = params;

    const accessTokenSignOptions = {
      algorithm: SIGN_ALGORITHM,
      expiresIn: expiresIn || "1d"
    } as jwt.SignOptions;

    const unsignedToken = {
      iss: issuer,
      sub: userRef,
      aud: audience,
      memberRef: memberRef,
      accessKeyId
    };

    return jwt.sign(unsignedToken, privateKey, accessTokenSignOptions);
  };
}
```

#### 2. 处理邀请接受

```typescript
function createUpdateMembershipStatus(identityConfig: IdentityConfig) {
  const prisma = createPrismaClient(
    identityConfig.dbUrl,
    identityConfig.encryptionKey
  );

  return async function updateMembershipStatus(token: string): Promise<void> {
    if (!isValidToken(token, identityConfig.privateKey)) {
      throw new Error("Invalid token");
    }

    const { memberRef } = jwtDecode(token) as { memberRef: string };

    await prisma.workspaceMember.update({
      where: {
        ref: memberRef
      },
      data: {
        status: WorkspaceMemberStatus.ACTIVE,
        updatedAt: new Date()
      }
    });
  };
}
```

## 安全特性

### 1. 数据加密

使用 Prisma 字段级加密保护敏感数据：

```prisma
model User {
  password String @map("password_hash") /// @encrypted
}

model ApiKey {
  accessKeySecret String @map("access_key_secret") /// @encrypted
}
```

### 2. 令牌轮换策略

- 访问令牌短期有效（通常 1 小时）
- 刷新令牌长期有效（通常 90 天）
- 支持令牌撤销和黑名单

### 3. 多因素认证 (MFA)

支持基于时间的一次性密码 (TOTP)：

```typescript
// 配置中启用 MFA
const identityConfig = {
  twoFactorAuthenticationRequired: true,
  // ... 其他配置
};
```

### 4. 联系方式验证

支持邮箱和电话号码验证：

```typescript
// 发送验证码
const sendResetPasswordCode = createSendResetPasswordCode(
  prisma,
  identityConfig
);

await sendResetPasswordCode({
  email: "user@example.com"
});
```

## 邮件和短信服务

### SMTP 邮件配置

```typescript
function createSendEmail(identityConfig: IdentityConfig) {
  const { smtpConfig } = identityConfig;
  const { host, port, secure, sender, auth } = smtpConfig;

  const transporter = nodemailer.createTransporter({
    host,
    port,
    secure,
    auth: {
      user: auth.user,
      pass: auth.pass
    }
  });

  return async function sendEmail(params: {
    to: string;
    subject: string;
    html: string;
  }) {
    return transporter.sendMail({
      from: sender,
      ...params
    });
  };
}
```

### Twilio 短信配置

```typescript
// 配置 Twilio SMS
const identityConfig = {
  twilioSmsConfig: {
    accountSid: "your_account_sid",
    authToken: "your_auth_token",
    sender: "+1234567890"
  }
};
```

## 开发指南

### 1. 本地开发

```bash
# 启动 Identity 服务
npm run start:identity

# 运行数据库迁移
npx prisma migrate dev

# 生成 Prisma 客户端
npx prisma generate
```

### 2. 环境变量配置

```bash
# 数据库配置
APISERVER_IDENTITY_DATABASE_URL="postgresql://user:password@localhost:5432/identity"

# JWT 配置
IDENTITY_ISSUER="https://identity.fonoster.com"
IDENTITY_AUDIENCE="api"
IDENTITY_PRIVATE_KEY_PATH="/path/to/private.pem"
IDENTITY_PUBLIC_KEY_PATH="/path/to/public.pem"

# 加密配置
IDENTITY_ENCRYPTION_KEY="your_encryption_key"

# SMTP 配置
IDENTITY_SMTP_HOST="smtp.gmail.com"
IDENTITY_SMTP_PORT="587"
IDENTITY_SMTP_USER="your_email@gmail.com"
IDENTITY_SMTP_PASS="your_app_password"

# OAuth2 配置
IDENTITY_GITHUB_CLIENT_ID="your_github_client_id"
IDENTITY_GITHUB_CLIENT_SECRET="your_github_client_secret"
```

### 3. 添加新的认证提供商

1. 在 `exchanges/` 目录下创建新的交换函数
2. 在 `utils/` 目录下添加提供商特定的工具函数
3. 更新 `IdentityConfig` 类型定义
4. 在服务构建器中注册新的交换函数

### 4. 自定义角色和权限

1. 更新 `Role` 枚举
2. 修改权限检查逻辑
3. 更新数据库迁移
4. 添加相应的测试

## API 接口

### gRPC 服务定义

```protobuf
service Identity {
  // 用户管理
  rpc CreateUser(CreateUserRequest) returns (CreateUserResponse);
  rpc GetUser(GetUserRequest) returns (GetUserResponse);
  rpc UpdateUser(UpdateUserRequest) returns (UpdateUserResponse);
  rpc DeleteUser(DeleteUserRequest) returns (DeleteUserResponse);
  
  // 工作空间管理
  rpc CreateWorkspace(CreateWorkspaceRequest) returns (CreateWorkspaceResponse);
  rpc GetWorkspace(GetWorkspaceRequest) returns (GetWorkspaceResponse);
  rpc UpdateWorkspace(UpdateWorkspaceRequest) returns (UpdateWorkspaceResponse);
  rpc ListWorkspaces(ListWorkspacesRequest) returns (ListWorkspacesResponse);
  
  // 令牌交换
  rpc ExchangeCredentials(ExchangeCredentialsRequest) returns (ExchangeResponse);
  rpc ExchangeApiKey(ExchangeApiKeyRequest) returns (ExchangeResponse);
  rpc ExchangeOauth2Code(ExchangeOauth2CodeRequest) returns (ExchangeResponse);
  rpc ExchangeRefreshToken(ExchangeRefreshTokenRequest) returns (ExchangeResponse);
  
  // 公钥获取
  rpc GetPublicKey(GetPublicKeyRequest) returns (GetPublicKeyResponse);
  
  // API 密钥管理
  rpc CreateApiKey(CreateApiKeyRequest) returns (CreateApiKeyResponse);
  rpc RegenerateApiKey(RegenerateApiKeyRequest) returns (RegenerateApiKeyResponse);
  rpc DeleteApiKey(DeleteApiKeyRequest) returns (DeleteApiKeyResponse);
  rpc ListApiKeys(ListApiKeysRequest) returns (ListApiKeysResponse);
}
```

## 故障排除

### 常见问题

1. **JWT 验证失败**
   - 检查公钥/私钥配置
   - 验证令牌格式和签名算法
   - 确认时钟同步

2. **数据库连接问题**
   - 检查连接字符串
   - 验证数据库权限
   - 确认网络连接

3. **OAuth2 认证失败**
   - 检查客户端 ID 和密钥
   - 验证回调 URL 配置
   - 确认授权范围

4. **邮件发送失败**
   - 检查 SMTP 配置
   - 验证邮箱凭据
   - 确认防火墙设置

### 调试技巧

```bash
# 启用详细日志
export LOGS_LEVEL=verbose

# 查看 JWT 令牌内容
echo "your_jwt_token" | base64 -d

# 测试数据库连接
npx prisma db pull

# 验证 SMTP 配置
npm run test:smtp
```

## 性能优化

### 1. 数据库优化

- 使用适当的索引
- 实施连接池
- 优化查询语句
- 定期清理过期数据

### 2. 缓存策略

- Redis 缓存用户会话
- 内存缓存公钥信息
- CDN 缓存静态资源

### 3. 令牌优化

- 合理设置令牌过期时间
- 实施令牌黑名单
- 使用短期访问令牌

## 扩展指南

### 1. 添加新的验证方式

1. 创建验证提供商接口
2. 实现具体的验证逻辑
3. 更新配置类型
4. 添加相应的测试

### 2. 集成新的通知渠道

1. 在 `utils/` 目录下创建通知函数
2. 更新配置以支持新渠道
3. 修改验证码发送逻辑
4. 添加错误处理

### 3. 自定义用户属性

1. 更新 Prisma schema
2. 创建数据库迁移
3. 修改用户管理函数
4. 更新 API 接口

Identity 模块是 Fonoster 系统安全的核心，提供了完整的身份管理解决方案。理解其架构和工作原理对于系统的安全性和可扩展性至关重要。