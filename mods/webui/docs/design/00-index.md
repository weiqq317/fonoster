# Fonoster WebUI 设计文档总览

本套文档基于对 `/mods/webui` 目录的源代码反向分析而成，覆盖整体架构、认证与会话、SDK 与上下文、页面与路由、工作区与资源管理、UI 架构、配置与构建、实例流程与故障排查等。所有文档均以可实施、可验证为目标，附带关键代码位置与示例说明。

文档目录：
- 01-overview.md：项目概览与目录结构
- 02-architecture.md：前端架构与模块划分
- 03-authentication.md：认证、OAuth、Cookie 与中间件
- 04-sdk.md：SDK 客户端扩展与刷新策略
- 05-contexts.md：全局上下文（Fonoster/Workspace）
- 06-ui.md：布局系统、导航与页面容器
- 07-workspaces.md：工作区模型与成员管理
- 08-resources.md：资源模块（Applications、SIP、Secrets、Storage、API Keys、Monitoring）
- 09-configuration.md：环境变量、构建与类型配置
- 10-examples.md：端到端示例流程（含代码入口）
- 11-troubleshooting.md：常见问题与排查建议

使用说明：按需阅读各章节，可直接在对应章节中找到具体代码文件路径与调用示例。