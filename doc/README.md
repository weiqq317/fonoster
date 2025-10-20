# Fonoster 项目开发文档

## 项目概述

Fonoster 是一个开源的可编程电信堆栈，旨在成为 Twilio 的开源替代方案。它允许企业通过基于云的实用程序将电话服务与互联网完全连接。

### 核心特性

- ✅ **多租户支持** - 支持多个组织和用户
- ✅ **PBX 功能部署** - 轻松部署专用交换机功能
- ✅ **可编程语音应用** - 灵活的语音应用开发
- ✅ **NodeJS SDK** - 完整的 JavaScript/TypeScript 开发工具包
- ✅ **Amazon S3 支持** - 集成 AWS 存储服务
- ✅ **Let's Encrypt 安全端点** - 自动 SSL 证书管理
- ✅ **OAuth2 认证** - 标准化身份验证
- ✅ **JWT 认证** - JSON Web Token 支持
- ✅ **基于角色的访问控制 (RBAC)** - 细粒度权限管理
- ✅ **插件化命令行工具** - 可扩展的 CLI 工具
- ✅ **Google Speech API 支持** - 语音识别和合成

## 项目架构

### 整体架构图

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Dashboard     │    │      CLI        │    │      SDK        │
│   (Web UI)      │    │   (fonoster)    │    │   (Node.js)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   API Server    │
                    │   (gRPC/HTTP)   │
                    └─────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         │                       │                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Identity      │    │     Voice       │    │   Autopilot     │
│   Service       │    │    Service      │    │   (AI Agent)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
                    ┌─────────────────┐
                    │   SIP Network   │
                    │    (Routr)      │
                    └─────────────────┘
                                 │
                    ┌─────────────────┐
                    │    Asterisk     │
                    │  (Media Server) │
                    └─────────────────┘
```

### 核心模块说明

#### 1. API Server (`mods/apiserver`)
- **功能**: 核心 API 服务器，提供 gRPC 和 HTTP 接口
- **技术栈**: Node.js, TypeScript, Prisma, gRPC
- **职责**: 
  - 统一 API 网关
  - 数据库操作
  - 业务逻辑处理
  - 服务编排

#### 2. Identity Service (`mods/identity`)
- **功能**: 身份认证和授权服务
- **技术栈**: Node.js, TypeScript, Prisma, JWT
- **职责**:
  - 用户认证
  - JWT 令牌管理
  - OAuth2 集成
  - 权限控制

#### 3. Voice Service (`mods/voice`)
- **功能**: 语音应用服务器
- **技术栈**: Node.js, TypeScript, gRPC
- **职责**:
  - 语音应用执行
  - 通话流程控制
  - 媒体处理指令

#### 4. Autopilot (`mods/autopilot`)
- **功能**: AI 驱动的智能助手
- **技术栈**: Node.js, TypeScript, LangChain, OpenAI
- **职责**:
  - 智能对话处理
  - 知识库集成
  - AI 语音助手

#### 5. Dashboard (`mods/dashboard`)
- **功能**: Web 管理界面
- **技术栈**: React, TypeScript, Material-UI, React Router
- **职责**:
  - 用户界面
  - 系统管理
  - 数据可视化

#### 6. SDK (`mods/sdk`)
- **功能**: 开发者工具包
- **技术栈**: TypeScript, gRPC, Rollup
- **职责**:
  - 客户端库
  - API 封装
  - 开发工具

#### 7. CLI Tool (`mods/ctl`)
- **功能**: 命令行管理工具
- **技术栈**: Node.js, TypeScript, oclif
- **职责**:
  - 命令行接口
  - 系统管理
  - 开发工具

### 支撑服务

#### 8. Common (`mods/common`)
- **功能**: 公共库和工具
- **职责**: 共享代码、工具函数、通用组件

#### 9. Types (`mods/types`)
- **功能**: TypeScript 类型定义
- **职责**: 类型安全、接口定义

#### 10. Logger (`mods/logger`)
- **功能**: 日志服务
- **职责**: 统一日志记录、日志聚合

#### 11. Streams (`mods/streams`)
- **功能**: 音频流处理
- **职责**: 实时音频流、媒体处理

#### 12. SIPNet (`mods/sipnet`)
- **功能**: SIP 网络栈
- **职责**: SIP 协议处理、网络通信

#### 13. Authorization (`mods/authz`)
- **功能**: 授权服务
- **职责**: 权限验证、访问控制

#### 14. MCP (`mods/mcp`)
- **功能**: 模型上下文协议
- **职责**: AI 模型集成、上下文管理

## 技术栈

### 后端技术
- **运行时**: Node.js 18+
- **语言**: TypeScript
- **框架**: Express.js, gRPC
- **数据库**: PostgreSQL (通过 Prisma ORM)
- **消息队列**: NATS
- **缓存**: Redis (可选)
- **监控**: InfluxDB

### 前端技术
- **框架**: React 19
- **语言**: TypeScript
- **UI 库**: Material-UI (MUI)
- **路由**: React Router v7
- **状态管理**: Zustand
- **数据获取**: TanStack Query

### 基础设施
- **容器化**: Docker, Docker Compose
- **代理**: Envoy Proxy
- **SIP 服务器**: Routr
- **媒体服务器**: Asterisk
- **RTP 引擎**: RTPEngine

### 开发工具
- **构建工具**: Lerna (Monorepo)
- **包管理**: npm
- **代码质量**: ESLint, Prettier
- **测试**: Mocha, Web Test Runner
- **CI/CD**: GitHub Actions

## 快速开始

### 环境要求
- Node.js 18+
- Docker & Docker Compose
- PostgreSQL 13+

### 安装步骤

1. **克隆项目**
```bash
git clone https://github.com/fonoster/fonoster.git
cd fonoster
```

2. **安装依赖**
```bash
npm install
```

3. **启动基础服务**
```bash
npm run start:services
```

4. **启动开发服务**
```bash
# 启动 API 服务器
npm run start:apiserver

# 启动 Dashboard
npm run start:dashboard

# 启动语音服务
npm run start:voice
```

## 文档导航

### 📚 开发文档
- [🚀 开发环境搭建](./development.md) - 开发环境配置、贡献指南和最佳实践
- [⚙️ 配置指南](./configuration.md) - 环境变量、服务配置和部署设置
- [🔧 故障排除](./troubleshooting.md) - 常见问题诊断和解决方案
- [📖 API 接口文档](./api.md) - 完整的 API 参考和使用示例

### 🏗️ 架构文档
- [📦 核心模块文档](./modules/README.md) - 各模块详细说明
- [🚀 部署指南](./deployment/README.md) - 生产环境部署

### 📋 快速链接
- [贡献指南](../CONTRIBUTING.md) - 如何为项目做出贡献
- [许可证](../LICENSE) - 项目许可证信息
- [更新日志](../CHANGELOG.md) - 版本更新记录

## 贡献指南

欢迎为 Fonoster 项目做出贡献！请参阅以下文档：

- [🚀 开发环境搭建](./development.md) - 了解如何设置开发环境
- [📋 贡献指南](../CONTRIBUTING.md) - 详细的贡献流程和规范
- [🔧 故障排除](./troubleshooting.md) - 开发过程中遇到问题的解决方案

## 许可证

本项目采用 MIT 许可证。详情请参阅 [LICENSE](../LICENSE) 文件。