# agents/ — Agent管理组件目录

> **一句话总结**：负责Claude Code智能体(Agent)的创建、编辑、查看和管理功能的UI组件集合。

---

## 目录职责

`agents/`目录包含管理Claude Code智能体（Agent）的完整功能组件。这些组件支持：
- 查看已存在的Agent列表和详情
- 创建新的Agent（通过向导或手动）
- 编辑Agent的配置（工具、模型、颜色等）
- 管理Agent的文件存储

Agent是Claude Code的扩展机制，允许用户定义具有特定系统提示、工具集和行为的自定义AI助手。

---

## 文件清单

### 核心组件文件

| 文件 | 职责简述 |
|------|---------|
| [AgentDetail.tsx](./AgentDetail.tsx.md) | Agent详情展示组件，显示单个Agent的完整信息 |
| [AgentEditor.tsx](./AgentEditor.tsx.md) | Agent编辑器，支持修改Agent的工具、模型和颜色 |
| [AgentsList.tsx](./AgentsList.tsx.md) | Agent列表组件，展示所有可用Agent |
| [AgentsMenu.tsx](./AgentsMenu.tsx.md) | Agent菜单主入口，管理整个Agent功能的导航 |
| [AgentNavigationFooter.tsx](./AgentNavigationFooter.tsx.md) | 导航底部栏，显示操作提示和退出状态 |

### 编辑器子组件

| 文件 | 职责简述 |
|------|---------|
| [ColorPicker.tsx](./ColorPicker.tsx.md) | 颜色选择器，用于设置Agent的背景色 |
| [ModelSelector.tsx](./ModelSelector.tsx.md) | 模型选择器，选择Agent使用的AI模型 |
| [ToolSelector.tsx](./ToolSelector.tsx.md) | 工具选择器，配置Agent可用的工具集 |

### 工具函数

| 文件 | 职责简述 |
|------|---------|
| [agentFileUtils.ts](./agentFileUtils.ts.md) | Agent文件操作工具，读写Agent定义文件 |
| [generateAgent.ts](./generateAgent.ts.md) | Agent生成器，使用AI自动生成Agent配置 |
| [types.ts](./types.ts.md) | Agent相关的类型定义 |
| [utils.ts](./utils.ts.md) | 通用工具函数 |
| [validateAgent.ts](./validateAgent.ts.md) | Agent配置验证逻辑 |

### 子目录

| 目录 | 职责简述 |
|------|---------|
| [new-agent-creation/](./new-agent-creation/README.md) | 新Agent创建向导的组件和步骤 |

---

## 内部协作关系

1. **AgentsMenu**是主入口，管理整个Agent功能的导航状态
2. **AgentsList**展示Agent列表，点击后进入**AgentDetail**查看详情
3. **AgentEditor**提供编辑功能，使用**ToolSelector**、**ModelSelector**、**ColorPicker**作为子组件
4. **new-agent-creation/**包含创建新Agent的分步向导
5. **agentFileUtils.ts**处理所有文件系统操作，与Agent存储交互

---

## 数据流

```
用户 -> AgentsMenu -> AgentsList -> AgentDetail -> AgentEditor
                       |
                       v
              new-agent-creation (向导)
                       |
                       v
              agentFileUtils (文件操作)
```
