# CLAUDE.md

请使用中文写提案和回答
这个文件为 Claude Code (claude.ai/code) 提供指导，用于处理此代码库中的代码。

TEngine 基于 HybridCLR + YooAsset + UniTask + Luban 构建。

---

## ⚡ 强制工作流（所有任务必须遵守）

> **禁止跳过** — 无论任务大小，必须按此顺序执行：

### 第一步：强制查询 Wiki（使用 wiki-query-agent）

**开始任何编码/方案/分析前，必须先调用 `wiki-query-agent` subagent 查询相关文档。**

```
规则：直接读取 wiki 文件 → ❌ 禁止
规则：通过 wiki-query-agent 查询 → ✅ 必须
```

**触发时机** — 以下任意情况立即调用 `wiki-query-agent`：
- 涉及任何 TEngine 模块（Resource/UI/Audio/Timer/Event/FSM...）
- 涉及热更代码（HybridCLR/GameApp）
- 涉及资源加载/释放（YooAsset）
- 涉及 UI 开发（UIWindow/UIWidget）
- 涉及事件系统（GameEvent/AddUIEvent）
- 涉及 Luban 配置表
- 涉及代码规范/架构设计
- 不确定某 API 用法时

**调用方式**：
```
使用 Agent 工具，subagent_type = "wiki-query-agent"
在 prompt 中描述需要查询的技术问题或功能点
```

**示例**：
```
// 用户要求实现背包 UI
→ 立即调用 wiki-query-agent，查询："UIWindow 开发规范、UIWidget 使用方式、资源加载释放"
→ 等待 subagent 返回结果
→ 基于结果编写代码
```

### 第二步：显式声明已查询文档

在输出代码/方案前，列出：
- 已通过 wiki-query-agent 查询的主题
- 关键规范摘要（来自 subagent 返回结果）

### 第三步：输出代码/方案

基于 wiki-query-agent 返回的规范编写实现。

---

## 📚 Wiki 知识库

> **唯一权威来源：`repowiki/zh/content/`**

Wiki 目录索引：[repowiki/zh/content/index.md](repowiki/zh/content/index.md)

**主要模块覆盖**：核心架构 / 模块系统 / 资源管理 / 热更新 / 事件系统 / UI系统 / 音频 / 本地化 / 流程管理 / 配置系统 / 内存管理 / 性能优化 / API参考

---

## 核心原则（编码红线）

1. **异步优先**：IO 操作用 `UniTask`，禁止同步加载/Coroutine
2. **模块访问**：通过 `GameModule.XXX` 访问，而非 `ModuleSystem.GetModule<T>()`
3. **资源必须释放**：`LoadAssetAsync` 对应 `UnloadAsset`，GameObject 用 `LoadGameObjectAsync`
4. **热更边界**：`GameScripts/Main` 不热更，`GameScripts/HotFix/` 全部热更
5. **事件解耦**：模块间用 `GameEvent`，UI 内部用 `AddUIEvent`

---

## wiki-query-agent 使用规范

### 为什么必须用 subagent
- wiki-query-agent 在独立上下文中运行，**不占用主 Agent 上下文窗口**
- 大量文档内容由 subagent 处理后，只返回精华摘要给主 Agent
- 保持主 Agent 上下文干净，专注于代码生成

### 查询时机（不可跳过）
| 场景 | 必须查询主题 |
|------|------------|
| UI 开发 | UIWindow 生命周期、UIWidget 规范、资源加载释放 |
| 资源加载 | LoadAssetAsync API、释放时机、YooAsset 规范 |
| 热更代码 | HybridCLR 程序集划分、GameApp 入口、热更边界 |
| 事件系统 | GameEvent 用法、AddUIEvent 规范、事件解耦模式 |
| 模块使用 | GameModule.XXX API、模块生命周期 |
| Luban 配置 | 配置表生成流程、访问方式 |
| 代码规范 | 命名约定、设计模式、架构约束 |

### 并行查询（多主题时）
若任务涉及多个主题，**并行启动多个 wiki-query-agent**：
```
同时查询 UI规范 + 资源管理规范 → 汇总后再编码
```

---

## 补充文档参考（技能文档）

详细技能文档见 `.claude/skills/tengine-dev/references/`（仅供 wiki-query-agent 内部查阅）：

| 文档 | 内容 |
|-----|------|
| architecture.md | 项目结构/启动流程 |
| modules.md | 模块 API（Timer/Scene/Audio/Fsm）|
| ui-development.md | UI 开发 |
| event-system.md | 事件系统 |
| resource-management.md | 资源加载 |
| hotfix-development.md | 热更代码 |
| luban-config.md | 配置表 |
| conventions.md | 代码规范 |
| troubleshooting.md | 问题排查 |
| unity-mcp-guide.md | MCP 工具索引 |
| ui-prefab-builder.md | UI Prefab 拼接 |
| scene-gameobject.md | 场景/GameObject 操作 |
| script-asset-workflow.md | 脚本/资源管理 |

---

## 🔧 自我优化机制

### 问题记录
发现问题时记录到 `.claude/memory/`：
- 文件名：`problem_YYYY-MM-DD.md`
- 内容：问题现象、原因分析、解决方案

### 文档同步
发现文档与代码不一致时：
```bash
/wiki:sync
```

### 优化触发条件
- 代码实现与 wiki 文档描述不符
- 发现框架潜在问题或 bug
- 文档缺失或过时
- 规范需要更新
