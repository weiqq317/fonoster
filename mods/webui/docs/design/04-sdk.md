# SDK 客户端扩展与刷新策略

文件与角色：
- `sdk/config/sdkConfig.ts`
- `sdk/hooks/useFonosterClient.tsx`
- `sdk/auth/AuthClient.ts`

扩展的 `WebClient` 接口：
- 访问密钥：`setAccessKeyId(accessKeyId, secret?)`
- 令牌读写：`getIdToken / setIdToken / setAccessToken / setRefreshToken`
- 兼容：若 SDK 缺少对应方法，`AuthClient` 通过对象属性回退设置。

`createFonosterClient` 工厂：
- 读取 `SDK_CONFIG`（如 `apiUrl`、令牌刷新阈值）
- 构造基础 Client 并注入扩展方法，供 `FonosterContext` 初始化。

`useFonosterClient` 钩子：
- 从 `FonosterContext` 读取 `client/isInitialized/session/authClient`。
- 暴露：
  - `authentication.signIn/signOut/executeWithRefresh/handleOAuth2Signup`
  - `verifyCode/sendVerificationCode`（客户端验证与下发验证码）
  - `setAccessKeyId`（在资源需要时设置访问密钥）
- 通知：统一通过 `useNotification` 捕获错误并展示。

刷新策略（在 `AuthClient` 中实现）：
- 前置检查：执行操作前判断 `cookieUtils.shouldRefreshAccessToken()`。
- 遇到令牌错误：`isTokenError()` 检测信息包含 `token expired/invalid token`，触发刷新与重试。
- 刷新失败：`handleAuthError()` 清理 Cookie 并触发登出。

示例：统一包裹资源请求
```ts
const { authentication } = useFonosterClient();

const listWorkspaces = async () => {
  return authentication.executeWithRefresh(async () => {
    const client = new Workspaces();
    // ...配置 client
    return client.listWorkspaces({ pageSize: 20 });
  });
};
```

建议：
- 将所有 SDK 请求统一在 `executeWithRefresh` 下执行，避免各模块重复处理令牌逻辑。
- 若后端增加新的令牌错误文案，需同步更新 `isTokenError()`。