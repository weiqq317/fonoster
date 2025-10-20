# 常见问题与排查建议

无法进入受保护页面：
- 检查 `refreshToken` 是否存在且未过期（`cookieUtils.hasValidAuthTokens`）。
- 确认中间件未误拦截公共路由（`/signin`、`/signup`、`/oauth/callback`）。

频繁要求验证邮箱/手机号：
- `idToken` 中 `emailVerified/phoneNumberVerified` 为 false 时中间件会强制 `/signup/verify`。
- 后端更新验证字段需确认 `idToken` 的声明（claims）一致。

令牌过期报错：
- 统一使用 `authentication.executeWithRefresh` 包裹；
- 若仍报错，检查 `AuthClient.isTokenError()` 文案匹配是否缺失；
- 确认 `tokenUtils.shouldRefreshToken()` 阈值是否满足业务需求（默认 5 分钟）。

OAuth 回调异常：
- 确认 `state` 解码成功且包含 `{ provider, action }`；
- 检查环境变量 `redirectUri/redirectUriCallback/scope` 是否配置完整；
- 回退策略：失败时 `router.push('/signin' | '/signup')`。

Cookie 设置失败：
- 开发环境不启用 `secure`；生产启用并验证跨域场景下的 `sameSite=lax`；
- 确认 `path=/` 与 `maxAge` 设置符合期望。

导航不显示工作区：
- 检查 `WorkspaceContext` 是否加载工作区列表并设置 `currentWorkspace`；
- 路由中是否包含 `[workspaceId]`，避免路径生成失败。

调试建议：
- 在浏览器中检查 Cookie 与本地存储（若使用），验证 `id/access/refresh` 三者是否正确写入与更新。
- 打印 `AuthClient.refreshSession()` 的异常，确认是网络错误还是令牌校验失败。