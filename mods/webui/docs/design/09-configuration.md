# 配置、环境变量与构建

环境变量（摘自 `README.md` 与 `src/config/oauth.ts`）：
- `NEXT_PUBLIC_FRONTEND_URL`：前端基准 URL
- `NEXT_PUBLIC_GITHUB_CLIENT_ID`：GitHub OAuth 客户端 ID
- `NEXT_PUBLIC_GITHUB_URL`：GitHub OAuth 授权端点
- `NEXT_PUBLIC_OAUTH_REDIRECT_URI`：OAuth 回调相对路径（拼接到前端 URL）
- `NEXT_PUBLIC_GITHUB_SIGNIN_REDIRECT_URI`、`NEXT_PUBLIC_GITHUB_SIGNUP_REDIRECT_URI`
- `NEXT_PUBLIC_GITHUB_SIGNIN_SCOPE`、`NEXT_PUBLIC_GITHUB_SIGNUP_SCOPE`

构建与脚本（`package.json`）：
- `dev`：开发服务器
- `build`：构建产物
- `storybook`：启动组件故事集（`@stories/*`）

TypeScript 配置（`tsconfig.json`）：
- 路径别名：`@/common/*`、`@/pages/*` 等
- 严格模式与项目引用确保类型校验与编译性能

Next/Vite 配置：
- `next.config.ts` 与 `vite.config.ts` 提供别名与开发服务器配置；优先以 Next 运行。

中间件与路由匹配：
- `export const config = { matcher: ["/((?!api|_next/static|_next/image|favicon.ico).*)"] }`
- 避免对静态资源与 API 路径进行认证拦截。

建议：
- 在 `.env.local` 中完整配置所有 OAuth 变量，确保 `signin/signup` 两条链路独立可测。
- 生产环境开启 `secure` Cookie，保证跨站重定向兼容（`sameSite=lax`）。