# 工作区模型与成员管理

核心钩子：`src/common/sdk/hooks/useWorkspaces.tsx`
- 能力：
  - CRUD：`listWorkspaces/createWorkspace/getWorkspace/updateWorkspace/deleteWorkspace`
  - 成员管理：`inviteUserToWorkspace/resendWorkspaceMembershipInvitation/removeUserFromWorkspace`
- 统一刷新：所有操作通过 `authentication.executeWithRefresh` 包裹，自动处理令牌刷新与错误通知。

上下文：`src/common/sdk/provider/WorkspaceContext.tsx`
- 维护：工作区列表、当前选中工作区、加载中状态
- 提供：选择工作区、刷新列表等方法，驱动导航与页面。

路由结构：`src/pages/workspace/[workspaceId]/*`
- 子模块：`overview`、`applications`、`sip-network`（`trunks/domains/credentials`）、`storage`、`secrets`、`api-keys`、`monitoring` 等。

示例：创建工作区
```ts
import { useWorkspaces } from "@/common/sdk/hooks/useWorkspaces";

const { createWorkspace } = useWorkspaces();
await createWorkspace({ name: "Demo Workspace" });
```

示例：邀请成员加入工作区
```ts
const { inviteUserToWorkspace } = useWorkspaces();
await inviteUserToWorkspace({ workspaceId, email: "user@example.com" });
```

示例：选择当前工作区（上下文）
```tsx
const { workspaces, currentWorkspace, selectWorkspace } = useWorkspaceContext();
selectWorkspace(workspaceId);
```

最佳实践：
- 在导航（`useNavConfig`）与页面加载时总是依赖当前工作区 ID，确保路径与数据一致。
- 成员邀请后可使用 `resendWorkspaceMembershipInvitation` 处理未达邮件的场景。