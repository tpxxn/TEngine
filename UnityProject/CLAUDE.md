# CLAUDE.md

请使用中文写提案和回答
这个文件为 Claude Code (claude.ai/code) 提供指导，用于处理此代码库中的代码。

TEngine 基于 HybridCLR + YooAsset + UniTask + Luban 构建。

---

## ⚡ 强制工作流（所有任务必须遵守）

> **禁止跳过** — 无论任务大小，必须按此顺序执行：

### 第零步：判断任务等级

在执行任何操作前，先判断任务等级：

| 等级 | 判断标准 | 知识查询策略 |
|------|---------|-------------|
| **L1 简单** | typo 修正、注释修改、日志输出、单行变量改名 | ❌ 跳过查询，直接编码 |
| **L2 调用** | 调用已知 API、单一模块的局部修改 | ✅ 触发 `tengine-dev` skill（只查该主题） |
| **L3 功能** | 新功能开发、跨文件修改、新增 UI/资源/事件逻辑 | ✅ 触发 `tengine-dev` skill（全量相关主题） |
| **L4 架构** | 模块设计、系统重构、多模块协作、架构决策 | ✅ 触发 `tengine-dev` skill（并行多主题） |

> **判断原则**：宁可高估等级，不可低估——不确定时上调一级。

---

### 第一步：按等级获取规范（使用 tengine-dev skill）

**L1 任务直接跳到第二步。L2-L4 必须先触发 `tengine-dev` skill。**

**知识源**：`.claude/skills/tengine-dev/references/`（AI 专用精炼文档，唯一权威来源）

#### 调用方式

```
使用 Skill 工具，skill = "tengine-dev"
描述需要查询的技术问题或功能点
```

#### 会话内缓存（避免重复查询）

同一会话中已查询过的主题无需重复触发 skill：
- 直接引用本次会话已获取的规范摘要
- 仅当任务涉及**本次会话未覆盖的新主题**时才重新触发

#### 触发时机

| 场景 | 必须查询主题 |
|------|------------|
| UI 开发 | ui-development.md — UIWindow 生命周期、UIWidget 规范 |
| 资源加载 | resource-management.md — LoadAssetAsync API、释放时机 |
| 热更代码 | hotfix-development.md — 程序集划分、GameApp 入口、热更边界 |
| 事件系统 | event-system.md — GameEvent 用法、AddUIEvent 规范 |
| 模块使用 | modules.md — GameModule.XXX API、模块生命周期 |
| Luban 配置 | luban-config.md — 配置表生成流程、访问方式 |
| 代码规范 | conventions.md — 命名约定、设计模式、架构约束 |

---

### 第二步：输出代码/方案

基于 tengine-dev skill 返回的规范编写实现。

**当 references 规范与代码实际 API 冲突时**：
1. 优先信任代码中的实际实现
2. 在输出中标注冲突点

---

## 核心原则（编码红线）

1. **异步优先**：IO 操作用 `UniTask`，禁止同步加载/Coroutine
2. **模块访问**：通过 `GameModule.XXX` 访问，而非 `ModuleSystem.GetModule<T>()`
3. **资源必须释放**：`LoadAssetAsync` 对应 `UnloadAsset`，GameObject 用 `LoadGameObjectAsync`
4. **热更边界**：`GameScripts/Main` 不热更，`GameScripts/HotFix/` 全部热更
5. **事件解耦**：模块间用 `GameEvent`，UI 内部用 `AddUIEvent`

---

## 📚 References 参考文档

> **AI 唯一权威来源：`.claude/skills/tengine-dev/references/`**

| 文档 | 内容 |
|-----|------|
| architecture.md | 项目结构/启动流程 |
| modules.md | 模块 API（Timer/Scene/Audio/Fsm）|
| ui-development.md | UI 开发 |
| event-system.md | 事件系统 |
| resource-management.md | 资源加载 |
| hotfix-development.md | 热更代码（HybridCLR/程序集划分）|
| hotpatch-management.md | 热更包下载/Manifest/缓存清理 |
| luban-config.md | 配置表 |
| conventions.md | 代码规范 |
| troubleshooting.md | 问题排查 |
| unity-mcp-guide.md | MCP 工具索引 |
| ui-prefab-builder.md | UI Prefab 拼接 |
| scene-gameobject.md | 场景/GameObject 操作 |
| script-asset-workflow.md | 脚本/资源管理 |
| material-shader-vfx.md | 材质/Shader/VFX/动画控制器 |
| editor-automation.md | Editor 自动化/PlayMode/测试/日志 |

---

## 🔧 自我优化机制

### 问题记录
发现问题时记录到 `.claude/memory/`：
- 文件名：`problem_YYYY-MM-DD.md`
- 内容：问题现象、原因分析、解决方案
