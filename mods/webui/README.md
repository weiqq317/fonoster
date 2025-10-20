# Fonoster WebUI

Fonoster WebUI 是 Fonoster 平台的现代化强大用户界面，基于 Next.js 和 Material UI 构建。该应用程序为管理 Fonoster 资源提供了直观的用户体验，具有强大的身份验证系统和可扩展的架构。

## 主要特性

- **完整身份验证**：支持凭据和 OAuth（GitHub）登录
- **会话管理**：自动处理 JWT 令牌和续期
- **SDK 集成**：与 Fonoster SDK 直接连接
- **现代界面**：基于 Material UI 的 UI 组件
- **响应式体验**：适配移动端和桌面设备的设计
- **Storybook 开发**：文档化的组件库

## 快速开始

1. 安装依赖：

```bash
npm install
# 或
yarn install
```

2. 配置环境变量：

```bash
# 复制示例文件
cp .env.example .env.local

# 根据您的环境编辑变量
```

3. 启动开发服务器：

```bash
npm run dev
# 或
yarn dev
```

4. 在浏览器中打开 [http://localhost:3000](http://localhost:3000)。

## 配置

### 环境变量

应用程序使用集中式配置系统。在您的 `.env.local` 中配置这些变量：

```env
# 前端 URL
NEXT_PUBLIC_FRONTEND_URL=http://localhost:3000

# OAuth 配置
NEXT_PUBLIC_OAUTH_REDIRECT_URI=/oauth/callback
NEXT_PUBLIC_GITHUB_CLIENT_ID=your_client_id_here
NEXT_PUBLIC_GITHUB_URL=https://github.com/login/oauth/authorize
NEXT_PUBLIC_GITHUB_SIGNIN_REDIRECT_URI=/workspace
NEXT_PUBLIC_GITHUB_SIGNIN_SCOPE=read:user
NEXT_PUBLIC_GITHUB_SIGNUP_REDIRECT_URI=/workspace
NEXT_PUBLIC_GITHUB_SIGNUP_SCOPE=user:email
```

## 开发工具

### 使用 Storybook 开发

要独立开发和测试组件：

```bash
npm run storybook
# 或
yarn storybook
```

打开 [http://localhost:6006](http://localhost:6006) 查看组件库。

## 贡献

欢迎贡献。请遵循以下步骤：

1. Fork 仓库
2. 为您的功能创建分支（`git checkout -b feature/amazing-feature`）
3. 提交您的更改（`git commit -m 'Add some amazing feature'`）
4. 推送到分支（`git push origin feature/amazing-feature`）
5. 打开 Pull Request

## 项目结构

```
fonoster-webui/
├── .next/              # Next.js 构建输出
├── .storybook/         # Storybook 配置
├── .vscode/           # VS Code 配置
├── docs/              # 项目文档
├── node_modules/      # 依赖项
├── public/            # 静态文件
├── src/
│   ├── common/        # 共享代码和组件
│   ├── config/        # 配置文件
│   ├── pages/         # Next.js 页面
│   │   ├── signin/    # 身份验证页面
│   │   ├── signup/    # 注册页面
│   │   ├── oauth/     # OAuth 处理
│   │   ├── workspace/ # 主应用程序工作区
│   │   ├── personal/  # 用户配置文件设置
│   │   ├── reset-password/ # 密码重置流程
│   │   ├── forgot-password/ # 密码恢复
│   │   ├── reset/     # 密码重置确认
│   │   ├── _app.tsx   # 主应用程序包装器
│   │   ├── _document.tsx # 自定义文档组件
│   │   ├── _error.tsx # 错误处理页面
│   │   └── 404.tsx    # 自定义 404 页面
│   ├── types/         # TypeScript 类型定义
│   ├── utils/         # 工具函数
│   └── middleware.ts  # Next.js 中间件
├── stories/           # Storybook 故事
├── theme/             # 主题配置
├── .env              # 环境变量
├── .env.example      # 示例环境变量
├── .env.local        # 本地环境变量
├── next.config.ts    # Next.js 配置
├── package.json      # 项目依赖和脚本
├── postcss.config.mjs # PostCSS 配置
├── tsconfig.json     # TypeScript 配置
└── vite.config.ts    # Vite 配置
```

## 核心概念

### 身份验证系统

```typescript
import { useFonosterClient } from '@/common/sdk/hooks/useFonosterClient';

function MyComponent() {
  const { 
    client, 
    isReady, 
    isAuthenticated, 
    authentication 
  } = useFonosterClient();

  // 使用客户端和身份验证方法
}
```

最佳实践：
1. 在使用客户端之前始终检查 `isReady`
2. 对于需要身份验证的操作使用 `authentication.executeWithRefresh`
3. 使用 try/catch 块处理错误
4. 检查 `isAuthenticated` 以确定用户是否已登录

### Next.js 集成

1. **基于文件系统的路由**：
   - `pages/` 中的每个文件自动成为一个路由
   - 像 `pages/about.tsx` 这样的文件生成 `/about` 路由
   - 像 `pages/blog/[slug].tsx` 这样的嵌套文件夹生成动态路由

2. **混合渲染**：
   - 静态站点生成（SSG）
   - 服务器端渲染（SSR）
   - 客户端渲染
   - 增量静态再生

3. **API 路由**：
   - `pages/api/` 中的无服务器 API 端点
   - 非常适合在不需要单独服务器的情况下创建 API

## 开发指南

### 表单创建指南

本指南解释了如何按照项目的标准结构创建表单。

#### 表单结构

```
src/pages/workspace/[workspaceId]/your-feature/
├── _components/
│   └── form/
│       └── YourFeatureForm.tsx    # 主表单组件
├── [id].tsx                       # 编辑页面
└── new.tsx                        # 创建页面
```

#### 可用组件

1. **布局组件**（`@/common/components/layout/pages`）：
   - `PageContainer`：所有表单的主包装器
   - `PageContainer.Header`：页面标题和操作
   - `PageContainer.Subheader`：表单描述
   - `PageContainer.ContentForm`：表单内容包装器

2. **表单字段**（`@/common/hooksForm/`）：
   - `InputContext`：文本输入字段
   - `SelectContext`：下拉选择字段
   - `TextAreaContext`：多行文本字段
   - `CheckboxContext`：复选框字段
   - `RadioContext`：单选按钮组
   - `DatePickerContext`：日期选择字段

3. **验证**（`@hookform/resolvers/zod`）：
   - 使用 Zod 进行模式验证
   - 位于每个表单组件中
   - 定义表单字段规则和类型

#### 最佳实践

1. **表单组织**：
   - 将表单保存在 `_components/form/` 目录中
   - 使用一致的命名约定
   - 将业务逻辑与 UI 分离

2. **状态管理**：
   - 跟踪加载、错误和成功状态
   - 处理表单提交状态
   - 管理 API 交互
   - 使用通知进行用户反馈（参见通知系统部分）

3. **用户体验**：
   - 显示清晰的验证消息
   - 提供加载指示器
   - 显示成功/错误通知
   - 启用表单导航

4. **类型安全**：
   - 使用 TypeScript 接口
   - 实现 Zod 模式
   - 导出表单数据类型

### 通知系统

项目提供了使用 `useNotification` 钩子显示通知的统一方式。

#### 使用示例：

```typescript
import { useNotification } from "@/common/hooks/useNotification";

function YourComponent() {
  const { 
    notifySuccess, 
    notifyError, 
    notifyWarning, 
    notifyInfo,
    NotificationComponent 
  } = useNotification();

  // 显示成功消息
  notifySuccess("操作成功完成");

  return (
    <>
      <NotificationComponent />
      {/* 您的组件内容 */}
    </>
  );
}
```

#### 可用方法：

1. **notifySuccess**：绿色背景，6 秒后自动隐藏
2. **notifyError**：处理带有代码和消息的错误对象
3. **notifyWarning**：黄色/橙色背景用于警告
4. **notifyInfo**：蓝色背景用于一般通知

#### 自定义选项：

```typescript
interface NotificationOptions {
  title?: string;              // 可选标题文本
  message: string;             // 主通知消息
  type?: "success" | "error" | "warning" | "info";
  duration?: number;           // 时间（毫秒）
  position?: {                 // 通知位置
    vertical: "top" | "bottom";
    horizontal: "left" | "center" | "right";
  };
  showIcon?: boolean;          // 显示/隐藏状态图标
  autoHide?: boolean;          // 自动关闭通知
  customStyle?: {              // 自定义样式
    backgroundColor?: string;
    color?: string;
  };
  onClose?: () => void;        // 关闭回调
  showCountdown?: boolean;     // 显示倒计时
  countdownDuration?: number;  // 倒计时时间（秒）
}
```

## 资源

- [Next.js 文档](https://nextjs.org/docs)
- [Next.js 教程](https://nextjs.org/learn-pages-router)
- [Next.js 仓库](https://github.com/vercel/next.js)
