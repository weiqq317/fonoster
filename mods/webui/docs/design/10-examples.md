# 端到端示例流程

示例一：凭据注册 → 登录 → 进入工作区
- 页面：`/signup`（表单校验 `zod`）→ `createUser` → 自动调用 `signIn`
- 中间件：检测已登录后从 `/` 重定向到 `/workspace`
- 代码参考：`src/pages/signup/index.tsx`、`src/middleware.ts`

示例二：GitHub OAuth 登录
- 页面：`/signin` 按钮生成 `state={ provider:GITHUB, action:signin }` 并跳转至 `authUrl`
- 回调：`/oauth/callback` 解析 `code/state`，调用 `authentication.signIn({ provider: GITHUB, oauthCode })`
- 完成：跳转至 `OAUTH_CONFIG.signin.redirectUri`
- 代码参考：`src/pages/signin/index.tsx`、`src/pages/oauth/callback.tsx`、`src/config/oauth.ts`

示例三：创建工作区并邀请成员
- Hook：`useWorkspaces`
- 代码：
```ts
const { createWorkspace, inviteUserToWorkspace } = useWorkspaces();
const ws = await createWorkspace({ name: "Team A" });
await inviteUserToWorkspace({ workspaceId: ws.id, email: "alice@example.com" });
```

示例四：管理 SIP 凭据
- 页面：`/workspace/[workspaceId]/sip-network/credentials/...`
- 代码：
```tsx
// 表单页中使用 PageContainer + useCredentialForm
<PageContainer>
  {/* Header/Subheader ... */}
  <ContentForm>
    <CredentialFormFields />
  </ContentForm>
</PageContainer>
```

示例五：统一的令牌刷新包装
```ts
const { authentication } = useFonosterClient();
await authentication.executeWithRefresh(async () => {
  const client = new Applications();
  return client.listApplications({ pageSize: 10 });
});
```

示例六：通知与错误提示
```ts
import { useNotification } from "@/common/hooks/useNotification";
const { notifySuccess, notifyError } = useNotification();
try {
  // 执行某操作
  notifySuccess("操作成功");
} catch (e) {
  notifyError(e);
}
```

备注：所有示例均需在已初始化的 `FonosterProvider` 与受保护路由环境下运行。