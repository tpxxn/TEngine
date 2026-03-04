# AI 开发工作流指南

> 本文档介绍如何结合 openspec 和 tengine-dev Skills 进行 TEngine 项目的 AI 辅助开发。

**更新时间**: 2026-03-05

---

## 目录

- [概述](#概述)
- [openspec 工作流](#openspec-工作流)
- [tengine-dev Skills](#tengine-dev-skills)
- [集成工作流](#集成工作流)
- [最佳实践](#最佳实践)

---

## 概述

TEngine 项目提供了一套完整的 AI 辅助开发工作流，结合了：

- **openspec**: 规范驱动的变更管理工具
- **tengine-dev**: Claude Code 专用的 TEngine 开发技能
- **Unity-MCP**: Unity Editor 自动化操作工具

这套工作流可以帮助开发者：
- 使用规范化的流程管理需求和变更
- 获取专业的 TEngine 开发指导
- 自动化 Unity Editor 操作

---

## openspec 工作流

### 什么是 openspec？

OpenSpec 是一个为AI编码助手设计的规范驱动开发工具，它通过轻量级的工作流程，确保人类开发者和AI助手在编写任何代码之前就能对需求达成明确共识。它通过以下方式帮助团队：

- 创建标准化的提案 (Proposal)
- 生成技术设计文档 (Design)
- 定义详细的规范 (Specs)
- 跟踪实施任务 (Tasks)

核心特点
- 🚀 轻量级：无需API密钥，最小化设置
- 🔄 现有项目优先：特别适合修改现有功能 (1→n)
- 📋 变更跟踪：提案、任务和规范差异的完整生命周期管理
- 🤖 AI工具集成：支持多种主流AI编码助手
2.为什么需要OpenSpec？
传统AI编码助手的问题
当您使用AI编码助手时，是否遇到过以下情况：
- ❌ AI根据模糊提示生成不符合需求的代码
- ❌ 遗漏重要功能要求
- ❌ 添加了不必要的功能
- ❌ 代码行为不可预测
OpenSpec的解决方案
OpenSpec通过规范驱动开发解决这些问题：
✅ 明确共识：在编码前确定所有要求
✅ 结构化变更：所有相关文档集中管理
✅ 可审查输出：AI根据明确规范生成代码
✅ 版本控制：完整追踪所有变更历史

### 安装

```bash
npm install -g @fission-ai/openspec@latest

cd your-project-directory

openspec init
```
初始化过程会：
1. 询问您使用的AI工具（Claude Code、Cursor等）
2. 自动配置相应的斜杠命令
3. 创建 openspec/ 目录结构

### 基础命令

```bash
# 新建议题
opsx:propose 新建议题我要做什么

# 开始执行
opex:apply 我还要附加什么条件

# 修改完成手动归档
opex:archive

# 获取指令帮助
openspec instructions <artifact-id> --change "my-feature"
```

### 推荐配合claude-mem 内有向量数据库长久记忆！

### 变更结构

每个变更包含以下 artifacts：

| Artifact | 描述 |
|----------|------|
| proposal.md | 变更提案，说明"为什么"和"做什么" |
| design.md | 技术设计文档，说明"如何实现" |
| specs/*.md | 详细规范，定义系统行为 |
| tasks.md | 实施任务清单 |

---

## tengine-dev Skills

### 什么是 tengine-dev？

`tengine-dev` 是 Claude Code 的技能，专门用于 TEngine 框架开发指导。

### 触发条件

在 TEngine 项目中编写或修改代码时，以下关键词会触发 tengine-dev 技能：

- **模块系统**: ResourceModule, AudioModule, TimerModule, GameModule
- **UI 开发**: UIWindow, UIWidget, UIModule
- **事件系统**: GameEvent, EventInterface, AddUIEvent
- **资源管理**: YooAsset, LoadAssetAsync, UnloadAsset
- **热更代码**: HybridCLR, GameApp, HotFix
- **配置表**: Luban, ConfigSystem

### 核心原则

tengine-dev 技能遵循以下核心原则：

1. **异步优先**: IO 操作用 `UniTask`，禁止同步加载/Coroutine
2. **模块访问**: 通过 `GameModule.XXX` 访问
3. **资源必须释放**: `LoadAssetAsync` 对应 `UnloadAsset`
4. **热更边界**: `GameScripts/Main` 不热更，`GameScripts/HotFix/` 全部热更
5. **事件解耦**: 模块间用 `GameEvent`，UI 内部用 `AddUIEvent`

### 程序集分层

```
GameScripts/Main/       → 主包（不热更）
GameScripts/HotFix/
  ├── GameProto/        → Luban 配置代码
  └── GameLogic/        → 业务逻辑（GameApp.cs 入口）
```

---

## 集成工作流

### 完整开发流程

```bash
# 1. 使用 openspec 创建新变更
opsx:propose "add-user-system"

# 2. 编写 proposal.md
# 3. 编写 design.md
# 4. 编写 specs/*.md
# 5. 编写 tasks.md

# 6. 开始实施
opsx:eapply "额外条件"

# 7. 使用 tengine-dev 技能获取开发指导
# 在 Claude Code 中描述你的需求，技能会自动加载

# 完成后归档
opsx:archive
```

### 与 Claude Code 配合

1. **需求分析阶段**: 使用 openspec 记录需求和规范
2. **代码开发阶段**: 在 Claude Code 中描述需求，tengine-dev 提供指导
3. **Unity 操作阶段**: 使用 Unity-MCP 自动化 Editor 操作

### 常用命令速查

| 操作 | 命令 |
|------|------|
| 创建变更 | `openspec new change "<name>"` |
| 查看状态 | `openspec status --change "<name>"` |
| 列出变更 | `openspec list` |
| 归档完成 | `openspec archive <name>` |

---

## 最佳实践

### 1. 需求定义

- 使用 openspec 明确记录需求
- 确保每个需求都有对应的测试场景

### 2. 代码规范

- 遵循 TEngine 核心原则
- 使用异步编程 (UniTask)
- 及时释放资源

### 3. 文档维护

- 保持 AI 开发工作流文档更新
- 记录使用过程中的问题和解决方案

### 4. 团队协作

- 统一使用 openspec 管理变更
- 定期回顾和改进工作流

---

## 相关文档

- [openspec 官方文档](https://github.com/openspec/openspec)
- [TEngine 框架文档](Books/0-介绍.md)
- [tengine-dev 技能参考](skills/tengine-dev/references/)

---

*如有问题，请提交 Issue 或联系维护者。*
