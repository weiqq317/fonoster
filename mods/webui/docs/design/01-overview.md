# 项目概览与目录结构

本项目为 Fonoster WebUI，基于 Next.js 与 React 构建，采用 MUI 进行 UI 组件开发，并通过 Fonoster SDK 与后端交互。整体设计围绕“认证、工作区选择、资源管理”三条主线展开，强调会话安全、可扩展的资源钩子和统一的页面布局模式。

技术栈与关键依赖：
- Next.js `15.x`、React `19`
- Material UI（MUI）`6.x`
- Fonoster SDK 与类型包：`@fonoster/sdk`、`@fonoster/types`
- 路由中间件、Cookies 管理：`next/server`、`cookies-next`
- 表单与校验：`react-hook-form`、`zod`

顶层结构（精选）：
- `src/common/sdk`：SDK 扩展、认证客户端、Hook 与 Provider
- `src/common/components/layout`：布局系统（认证/未认证）、导航、页面容器
- `src/pages`：路由页面（认证、OAuth 回调、工作区模块）
- `src/config/oauth.ts`：OAuth 配置（GitHub 等）
- `src/middleware.ts`：路由访问控制中间件
- `src/common/utils`：Cookie/JWT 工具

核心理念：
- 认证优先：进入应用前强制会话检查与邮箱/手机号验证流程
- 统一数据访问：所有资源通过 `useFonosterClient` 与 `executeWithRefresh` 统一处理令牌刷新
- 页面标准化：PageContainer 提供统一的列表/表单/详情页面骨架

示例：入口与重定向
- 代码位置：`src/middleware.ts`
- 逻辑要点：
  - 根路径 `/` 根据是否已登录重定向到 `/workspace` 或 `/signin`
  - 非公共路由在未认证时统一重定向到 `/signin`

公共/受保护路由：
- 公共：`/signin`、`/signup`、`/forgot-password`、`/reset-password`、`/oauth/callback`
- 受保护：工作区及所有业务模块（需有效 Refresh Token）

环境变量（精选）：
- `NEXT_PUBLIC_FRONTEND_URL`：前端基准 URL
- `NEXT_PUBLIC_GITHUB_CLIENT_ID`、`NEXT_PUBLIC_GITHUB_URL`、各 `SIGNIN/SIGNUP` 的 `redirectUri/scope`

更多细节：见后续各章节。