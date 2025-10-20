# 认证、OAuth、Cookie 与中间件

核心组件与文件：
- 中间件：`src/middleware.ts`
- 认证客户端：`src/common/sdk/auth/AuthClient.ts`
- Cookie 工具：`src/common/utils/cookieUtils.ts`
- JWT 工具：`src/common/utils/tokenUtils.ts`
- OAuth 配置与类型：`src/config/oauth.ts`、`src/types/oauth.ts`
- OAuth 回调页：`src/pages/oauth/callback.tsx`

路由访问控制（中间件）：
- 忽略静态与 API 路径；其余路径强制检查 `refreshToken`。
- 未认证：非公共路径统一重定向到 `/signin`。
- 已认证：若 `idToken` 中 `emailVerified/phoneNumberVerified` 为 false，则重定向到 `/signup/verify`。
- 根路径：根据认证态重定向至 `/workspace` 或 `/signin`。

令牌与 Cookie：
- Cookie 名称：`idToken`、`accessToken`、`refreshToken`。
- 保存策略：`cookieUtils.saveAuthTokens()` 统一设置（`secure` 生产启用、`sameSite=lax`）。
- 清理策略：`cookieUtils.clearAuthTokens()` 在登出或错误态清理。
- 刷新判断：`cookieUtils.shouldRefreshAccessToken()` → `tokenUtils.shouldRefreshToken(accessToken)`。

JWT 工具关键逻辑：
- `isTokenExpired(token)`：过期或非法返回 true。
- `shouldRefreshToken(token, thresholdMinutes=5)`：距离过期阈值内返回 true。
- `decodeToken(token)`：解析并返回 `exp/sub/...`；异常返回 null。

认证客户端（主要流程）：
- `signIn(options)`：按 `AuthProvider` 分派到 `client.login()` 或 `client.loginWithOauth2Code()`；成功后 `saveTokensAndUpdateSession()`。
- `signUp(options)`：复用 `signIn`（支持 OAuth 注册）。
- `signOut()`：调用 SDK `logout()`（若存在），清理 Cookie，回调 `onSignOut()`，并跳转 `/signin`。
- `refreshSession()`：在需要刷新时请求服务端刷新，失败则 `handleAuthError()`（清 Cookie 并登出）。
- `executeWithRefresh(operation)`：在执行前后判断刷新需求，遇到令牌错误时尝试刷新并重试。
- `handleOAuth2Signup(tokens)`：用于前端直拿到 OAuth2 令牌的场景，写入 Client 与 Cookie，并置为已认证。

OAuth（以 GitHub 为例）：
- 配置来源：`src/config/oauth.ts` 读取环境变量，分别配置 `signin` 与 `signup` 的 `redirectUri/scope`。
- 发起：`/signin` 与 `/signup` 页面生成 `state`（含 `provider` 与 `action`），跳转至 `authUrl`。
- 回调：`/oauth/callback` 解析 `code` 与 `state`，根据 `action`：
  - `signin`：调用 `authentication.signIn({ provider: GITHUB, oauthCode: code })`，完成后跳转 `signin` 的 `redirectUri`。
  - `signup`：调用 `createUserWithOauth2Code(code)` 完成注册，跳转 `signup` 的 `redirectUri`。

实例：凭据登录
- 入口：`src/pages/signin/index.tsx`
- 表单校验：`zod` + `react-hook-form`
- 提交：`authentication.signIn({ provider: credentials, credentials: { username, password } })`
- 成功：中间件检测到认证态后进入工作区路由。

实例：OAuth 登录
- 入口：`src/pages/signin/index.tsx` → GitHub 按钮
- 回调处理：`src/pages/oauth/callback.tsx`，完成后跳转至 `OAUTH_CONFIG.signin.redirectUri`

安全建议：
- 生产环境启用 `secure` Cookie；`sameSite=lax` 保持重定向兼容。
- 避免将敏感信息存入 `idToken`，仅用于 UI 状态（邮箱/手机号验证）。