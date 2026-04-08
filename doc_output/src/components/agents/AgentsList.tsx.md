# AgentsList.tsx — Agent列表展示组件

> **一句话总结**：展示Agent列表，支持分类显示、键盘导航和创建新Agent入口。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/AgentsList.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 渲染可交互的Agent列表界面 |

---

## 功能概述

`AgentsList`组件用于展示所有可用的Agent，并提供交互功能让用户选择Agent或创建新Agent。它支持以下功能：
- 按来源分组显示Agent（内置、用户设置、项目设置等）
- 显示Agent的模型和内存配置信息
- 标识被覆盖/隐藏的Agent
- 键盘导航支持（上下箭头、回车选择）
- "Create new agent"快捷入口

---

## 核心内容详解

### 主要类型

- **Props**: 组件属性
  - `source`: Agent来源筛选（all、built-in、userSettings等）
  - `agents`: ResolvedAgent数组
  - `onBack`: 返回回调
  - `onSelect`: 选择Agent回调
  - `onCreateNew`: 创建新Agent回调（可选）
  - `changes`: 变更提示信息数组

### 主要函数

- **AgentsList**: 主组件函数
  - **状态管理**:
    - `selectedAgent`: 当前选中的Agent
    - `isCreateNewSelected`: 是否选中"创建新Agent"选项
  - **排序**: 使用`compareAgentsByName`对Agent排序
  - **筛选**: 根据source筛选可显示的Agent

- **renderAgent**: 渲染单个Agent项
  - 显示Agent名称、模型、内存信息
  - 标识内置Agent和已被覆盖的Agent
  - 显示警告图标表示被shadowed的Agent

- **handleKeyDown**: 键盘事件处理
  - 上下箭头导航
  - 回车键选择
  - 支持在"创建新Agent"和Agent列表间切换

### 渲染逻辑

1. **创建新Agent选项**: 如果有`onCreateNew`回调，显示创建入口
2. **可选项Agent列表**: 根据source分组显示非内置Agent
3. **内置Agent区域**: 单独显示内置Agent（不可选择）
4. **变更提示**: 使用Dialog显示变更信息

---

## 设计要点

- 使用React Compiler优化渲染性能
- 内置Agent只显示不可选择，避免误操作
- 支持显示Agent覆盖关系（shadowed by）
- 智能默认选中逻辑（优先选中"创建新Agent"或第一个Agent）

---

## 与其他文件的关系

- **依赖**:
  - `AgentNavigationFooter` - 导航提示
  - `Dialog`, `Divider` - UI组件
  - `resolveAgentModelDisplay` - 模型显示解析
  - `compareAgentsByName` - Agent排序
- **被依赖**:
  - `AgentsMenu` - 作为列表视图使用

---

## 注意事项

- 内置Agent（built-in）不可选择和编辑
- 被覆盖的Agent会显示警告图标和覆盖来源
- 支持显示Agent的模型和内存配置信息
