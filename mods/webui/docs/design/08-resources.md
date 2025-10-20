# 资源模块（Applications、SIP、Secrets、Storage、API Keys、Monitoring）

统一模式：
- 每个模块提供对应 Hook（如 `useApplications/useTrunks/useDomains/useCredentials` 等），内部：
  - 构造 `@fonoster/sdk` 客户端实体（`Applications/Trunks/Domains/Credentials/...`）
  - 通过 `useFonosterClient` 获取 `client/authentication`
  - 使用 `authentication.executeWithRefresh` 包裹所有 SDK 请求
  - 错误通过 `useNotification` 统一上报
  - 列表查询支持 `usePaginatedData` 进行分页、排序与过滤

SIP Network：
- 路由：`/workspace/[workspaceId]/sip-network/` 下含 `trunks/domains/credentials` 子模块
- 示例：`credentials/_components/form/CredentialForm.tsx` 展示表单页集成 `PageContainer` 与自定义 `useCredentialForm`

Secrets：
- 管理密钥与机密，建议配合 `setAccessKeyId`（来自 `useFonosterClient`）在敏感操作前设置访问密钥。

Storage：
- 对象存储相关操作，遵循统一的 Hook 模式与分页机制。

API Keys：
- 创建与管理访问密钥，确保在 `AuthClient` 与 Client 中令牌状态正确；必要时在页面入口设置 `accessKeyId`。

Monitoring：
- 监控与告警页面，数据展示遵循 PageContainer 的列表模式；通知通过 `useNotification`。

示例：列出 Trunks（分页）
```ts
const { listItems } = usePaginatedData<Trunk>({ pageSize: 20 });
const trunks = await authentication.executeWithRefresh(() => client.listTrunks({ cursor }));
```

建议：
- 资源模块新增时复制既有模式，并在 `nav/useNavConfig.ts` 与 `nav/utils/paths.ts` 增加导航项与路径函数。
- 所有网络操作必须走 `executeWithRefresh`，避免令牌过期导致的隐式失败。