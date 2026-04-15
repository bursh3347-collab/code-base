[English](README.md) | [中文](README_CN.md)

# 🔧 通用代码库 — 跨领域可复用代码模式

> **从全球50+高星开源项目中提取的最佳实践和可复用代码模式。**

这不是一个链接收藏夹。每个模块都包含：
- 🔍 **来源追溯** — 代码来自哪里（附GitHub链接和Stars数）
- 💡 **为什么好** — 这种方案比替代方案好在哪里
- 📖 **怎么用** — 详细使用说明和示例
- ✏️ **改了什么** — 与原项目的区别
- 📜 **许可证** — 原项目的开源协议

## 📦 模块

| 模块 | 描述 | 状态 |
|------|------|------|
| [auth/](./auth/) | 认证模式（JWT、OAuth、Magic Link） | 🟡 脚手架 |
| [database/](./database/) | 数据库最佳实践（Supabase、Prisma、Drizzle） | 🟡 脚手架 |
| [api/](./api/) | API设计模式（REST、GraphQL、tRPC） | 🟡 脚手架 |
| [ui/](./ui/) | UI组件模式（Shadcn、Tailwind） | 🟡 脚手架 |
| [deploy/](./deploy/) | 部署方案（Vercel、Docker、CI/CD） | 🟡 脚手架 |
| [ai-integration/](./ai-integration/) | AI集成模块（OpenAI、RAG、Embedding） | 🟡 脚手架 |
| [testing/](./testing/) | 测试策略与模式 | 🟡 脚手架 |
| [monitoring/](./monitoring/) | 监控与可观测性 | 🟡 脚手架 |
| [error-handling/](./error-handling/) | 错误处理模式 | 🟡 脚手架 |
| [logging/](./logging/) | 日志方案 | 🟡 脚手架 |
| [scheduling/](./scheduling/) | 调度与定时任务模式 | 🟡 脚手架 |
| [messaging/](./messaging/) | 消息与网关架构 | 🟡 脚手架 |

## 🚀 快速开始

1. **浏览模块** — 每个文件夹都是独立模块，有自己的README
2. **阅读README** — 了解模块内容和推荐方案
3. **复制所需** — 每个模块都为复制粘贴复用而设计
4. **检查SOURCES.md** — 使用前确认许可证

## 🏗️ 架构理念

- **松耦合** — 每个模块独立，无跨模块依赖
- **复制友好** — 文件级复制，非包管理
- **多选方案** — 每个模块提供2-3种方案并给出明确推荐
- **来源可追溯** — 每段代码都链接回原项目

## 🔄 构建方式

本仓库由 **天工情报系统** 持续供给：
- 🐙 **GitHub代码猎手** 每日扫描趋势仓库
- 🧭 **内阁首辅** 识别跨领域可复用模式
- 模式被提取、文档化后自动推送至此

## 📊 相关仓库

| 分类 | 仓库 | 成熟度 |
|------|------|--------|
| 🤖 AI Agent | [ai-agent-stack](https://github.com/bursh3347-collab/ai-agent-stack) | L2 成长期 |
| 🏗️ 全栈SaaS | [fullstack-saas-stack](https://github.com/bursh3347-collab/fullstack-saas-stack) | L1 成长期 |
| 🎨 前端 | [frontend-stack](https://github.com/bursh3347-collab/frontend-stack) | L1 成长期 |
| 🔧 通用代码库 | **code-base**（本仓库） | **L1 脚手架** |

## 📄 许可证

本仓库采用MIT许可证。各代码片段保留其原始许可证（详见 [SOURCES.md](./SOURCES.md)）。

---

*由天工情报系统驱动 — AI驱动的开源知识引擎*
