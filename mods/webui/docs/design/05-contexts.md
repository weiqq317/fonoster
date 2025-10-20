# 全局上下文（Fonoster/Workspace）

FonosterContext（`src/common/sdk/provider/FonosterContext.tsx`）：
- 职责：
  - 初始化 `WebClient`（`createFonosterClient`）
  - 提供 `authClient`（基于 `AuthClient`）与 `session`（`isAuthenticated`）
  - 提供 `setSession` 方法以供认证流更新会话态
- 消费：`useFonosterClient` 钩子读取并下发给资源模块。

WorkspaceContext（`src/common/sdk/provider/WorkspaceContext.tsx`）：
- 职责：
  - 加载工作区列表、当前选中项与加载态
  - 提供选择/刷新接口，驱动导航与页面数据
- 依赖：`useFonosterClient`、`useWorkspaces` 钩子。

上下文交互流程：
1. `FonosterProvider` 启动时创建 SDK Client 与 AuthClient，读取 Cookie 初始化 `session`。
2. 页面进入受保护路由后，中间件与 `FonosterContext.session` 一致性保证认证态。
3. `WorkspaceProvider` 在认证后加载工作区列表，`useNavConfig` 基于当前工作区生成侧栏路径。

示例：在页面中消费上下文
```tsx
import { useFonosterClient } from "@/common/sdk/hooks/useFonosterClient";
import { useWorkspaceContext } from "@/common/sdk/provider/WorkspaceContext";

const OverviewPage = () => {
  const { isAuthenticated } = useFonosterClient();
  const { currentWorkspace, listWorkspaces } = useWorkspaceContext();
  // 根据当前工作区渲染内容
};
```

最佳实践：
- 所有业务钩子从 `useFonosterClient` 获取认证与 Client，不直接访问 Cookie。
- 页面只读取上下文暴露的状态与操作，避免重复实现数据请求与刷新逻辑。