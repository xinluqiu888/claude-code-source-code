# AgentProgressLine.tsx — Agent 进度行显示组件

> **一句话总结**：在终端树状结构中渲染单个 Agent 的运行状态、工具调用次数和 Token 消耗。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/AgentProgressLine.tsx` |
| 文件类型 | TSX |
| 代码行数 | 135 |
| 主要职责 | 以树状缩进格式显示单个 Agent 的类型、描述、状态、工具使用次数和 Token 用量 |

---

## 功能概述

`AgentProgressLine` 组件负责在终端 UI 中渲染一个 Agent 的进度信息行。它使用 Unicode 树状字符（`├─` / `└─`）来表示层级关系，直观地展示多个 Agent 的树形结构。

组件会根据 Agent 的状态动态显示不同内容：正在运行时显示最后一次工具调用信息或"Initializing..."；后台运行（异步且已解决）时显示任务描述；完成时显示"Done"。

对于非后台运行的 Agent，还会在名称后附加工具调用次数（"N tool uses"）和 Token 消耗量。颜色显示通过主题系统的 `keyof Theme` 类型进行控制，支持背景色和前景色分别配置。

---

## 核心内容详解

### 导入与依赖

| 模块 | 来源 |
|------|------|
| `Box`, `Text` | `../ink.js`（终端 UI 原语） |
| `formatNumber` | `../utils/format.js`（数字格式化） |
| `Theme` | `../utils/theme.js`（主题类型） |

### 主要类/函数/接口

**Props 类型**：

| 属性 | 类型 | 说明 |
|------|------|------|
| `agentType` | `string` | Agent 类型标识 |
| `description` | `string?` | Agent 的描述文本 |
| `name` | `string?` | Agent 名称（hideType 模式下优先显示） |
| `descriptionColor` | `keyof Theme?` | 描述文字的背景色主题键 |
| `taskDescription` | `string?` | 后台运行时显示的任务描述 |
| `toolUseCount` | `number` | 已调用工具次数 |
| `tokens` | `number \| null` | 已消耗 Token 数 |
| `color` | `keyof Theme?` | Agent 类型标签的背景色主题键 |
| `isLast` | `boolean` | 是否为最后一个子节点（影响树状字符） |
| `isResolved` | `boolean` | 是否已完成 |
| `isError` | `boolean` | 是否出错（当前不影响渲染，接收但未使用） |
| `isAsync` | `boolean?` | 是否为异步/后台 Agent |
| `shouldAnimate` | `boolean` | 是否需要动画（接收但未使用） |
| `lastToolInfo` | `string \| null?` | 最后一次工具调用的信息 |
| `hideType` | `boolean?` | 是否隐藏 Agent 类型标签 |

**`getStatusText()` 函数**：根据 `isResolved` 和 `isBackgrounded` 返回状态文字：
- 未完成：`lastToolInfo` 或 `"Initializing…"`
- 后台运行：`taskDescription` 或 `"Running in the background"`
- 已完成：`"Done"`

### 数据流与逻辑流程

```
Props
  ├─ treeChar = isLast ? "└─" : "├─"
  ├─ isBackgrounded = isAsync && isResolved
  ├─ getStatusText() → 状态文字
  └─ 渲染结构：
       Box（column）
         ├─ Box（左缩进3）
         │    ├─ Text（树状字符）
         │    └─ Text（Agent 名称/类型 + 工具次数 + Token）
         └─ Box（状态行，后台运行时隐藏）
              └─ Text（⏿ 状态文字）
```

### 对外接口（导出内容）

- `export function AgentProgressLine(props: Props): React.ReactNode`

---

## 设计要点

- **树状字符**：使用 Unicode 字符 `├─`（`\u251C\u2500`）和 `└─`（`\u2514\u2500`）实现视觉层级。
- **hideType 模式**：当 `hideType=true` 时，优先显示 `name > description > agentType`，适用于不需要展示类型标签的场景。
- **颜色反转**：当提供颜色时，使用 `color="inverseText"` 实现彩色背景标签效果。
- **后台 Agent**：`isBackgrounded`（异步且完成）时隐藏工具调用统计和状态行，只显示任务描述。

---

## 与其他文件的关系

- **依赖**：
  - `../ink.js` — 终端 UI 组件
  - `../utils/format.js` — 数字格式化
  - `../utils/theme.js` — 主题类型定义
- **被依赖**：用于显示多 Agent 进度树的父组件（如 `CoordinatorAgentStatus.tsx` 等）

---

## 注意事项

- `isError` 和 `shouldAnimate` 属性当前在渲染逻辑中未被使用，可能是预留接口或由父组件在条件渲染层处理。
- 组件经过 React Compiler 优化，有 32 个记忆化槽位，修改时注意保持 props 引用稳定性。
