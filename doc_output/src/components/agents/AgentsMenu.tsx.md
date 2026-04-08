# AgentsMenu.tsx — Agent菜单主控制器

> **一句话总结**：Agent功能的中央控制器，管理不同模式下的视图切换和状态流转。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/AgentsMenu.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 协调Agent相关的所有功能和视图状态 |

---

## 功能概述

`AgentsMenu`是Agent功能的中央控制器组件，负责：
- 管理不同的工作模式（列表、详情、编辑、创建、删除确认）
- 处理Agent的CRUD操作
- 协调各个子组件之间的数据流
- 维护变更历史记录
- 处理退出逻辑

组件使用状态机模式管理复杂的UI状态流转，支持在列表、详情、编辑、创建等视图间无缝切换。

---

## 核心内容详解

### 主要类型

- **Props**:
  - `tools`: 可用的工具集
  - `onExit`: 退出回调，可携带结果信息

- **ModeState**: 模式状态联合类型
  - `list-agents`: 显示Agent列表
  - `view-agent`: 查看Agent详情
  - `edit-agent`: 编辑Agent
  - `create-agent`: 创建新Agent
  - `delete-confirm`: 删除确认

### 主要函数

- **AgentsMenu**: 主组件函数
  - **状态管理**:
    - `modeState`: 当前模式状态
    - `changes`: 变更历史记录
  - **数据获取**: 从AppState获取Agent定义和工具

- **handleAgentCreated**: Agent创建成功处理
  - 添加变更记录
  - 返回列表视图

- **handleAgentDeleted**: Agent删除处理
  - 调用`deleteAgentFromFile`删除文件
  - 更新全局AppState
  - 添加变更记录

### 模式处理逻辑

组件使用switch语句处理不同模式：
1. **list-agents**: 渲染`AgentsList`组件
2. **view-agent**: 渲染`AgentDetail`组件
3. **edit-agent**: 渲染`AgentEditor`组件
4. **create-agent**: 渲染`CreateAgentWizard`组件
5. **delete-confirm**: 渲染确认删除对话框

### Agent来源分组

支持按来源筛选Agent：
- `built-in`: 内置Agent
- `userSettings`: 用户设置
- `projectSettings`: 项目设置
- `policySettings`: 策略设置
- `localSettings`: 本地设置
- `flagSettings`: 命令行参数
- `plugin`: 插件Agent

---

## 设计要点

- 使用React Compiler进行渲染优化
- 状态机模式管理复杂的视图流转
- 集中处理Agent的CRUD操作
- 维护变更历史供用户查看

---

## 与其他文件的关系

- **依赖**:
  - `AgentsList`, `AgentDetail`, `AgentEditor` - 各模式视图
  - `CreateAgentWizard` - 创建向导
  - `deleteAgentFromFile` - 文件删除操作
  - `useAppState`, `useSetAppState` - 全局状态管理
- **被依赖**:
  - 作为Agent功能的主入口被外部调用

---

## 注意事项

- 内置Agent和插件Agent不能删除
- 所有变更都会记录到changes数组中显示给用户
- 使用useExitOnCtrlCDWithKeybindings处理退出快捷键
