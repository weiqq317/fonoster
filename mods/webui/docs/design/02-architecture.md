# 前端架构与模块划分

总览：
- 分层：`SDK 扩展/认证客户端` → `全局上下文` → `业务资源 Hooks` → `页面与布局`
- 关键管道：认证态与令牌在 `FonosterContext + AuthClient` 中统一管理，通过 `useFonosterClient` 下发至各业务 Hook。

模块角色：
- `sdk/config/sdkConfig.ts`：扩展 `WebClient` 接口，增加 `setAccessKeyId / getIdToken / setIdToken` 等方法；提供 `createFonosterClient` 工厂与 `SDK_CONFIG`（API URL、刷新阈值等）。
- `sdk/auth/AuthClient.ts`：封装登录/登出、令牌刷新、OAuth2 注册处理、错误重试；统一 Cookie 与 Client 的令牌写入。
- `sdk/provider/FonosterContext.tsx`：初始化 SDK Client，提供 `authClient` 与会话态；App 级别入口。
- `sdk/provider/WorkspaceContext.tsx`：维护当前工作区、列表与选择逻辑；驱动导航与页面数据查询。
- `sdk/hooks/*`：资源操作（Users、Workspaces、Domains、Trunks、Applications、Credentials 等），统一通过 `authentication.executeWithRefresh` 包裹请求。
- `common/components/layout/*`：认证/未认证布局、导航组件、页面容器等 UI 架构件。
- `middleware.ts`：请求访问控制与认证态路由重定向。

数据流与会话：
- 登录成功 → `AuthClient.saveTokensAndUpdateSession()` 将令牌写入 Cookie 与 Client，并更新 `FonosterContext.session`。
- 页面/Hook 调用 → 使用 `useFonosterClient.authentication.executeWithRefresh` 自动在过期前刷新令牌；失败时触发统一的错误处理（清 Cookie、登出）。

刷新与重试策略：
- `cookieUtils.shouldRefreshAccessToken()` 基于 `tokenUtils.shouldRefreshToken()` 判断是否接近过期。
- 若服务端返回令牌错误（过期/无效），`AuthClient.retryOperationWithFreshToken()` 先刷新再重试。

页面架构：
- 未认证页（`signin`/`signup` 等）使用 `noAuth/Layout`，居中卡片，便于表单交互。
- 认证页使用 `auth/Layout` 与 `nav/index.tsx` 的 `SecuredLayout`，包含头部、移动/桌面侧栏与主内容区；侧栏项目由 `useNavConfig` 动态生成，依赖当前工作区。

示例代码入口：
- 认证入口：`src/pages/signin/index.tsx`、`src/pages/signup/index.tsx`
- OAuth 回调：`src/pages/oauth/callback.tsx`
- 工作区页：`src/pages/workspace/[workspaceId]/overview/index.tsx`（及同级模块目录）

扩展性：
- 新资源模块按既有 Hook 模式复制：定义类型入参、CRUD 方法、统一分页（`usePaginatedData`）与通知（`useNotification`）。
- 导航通过 `useNavConfig` 增加条目，同时在 `paths.ts` 中提供路由生成函数。