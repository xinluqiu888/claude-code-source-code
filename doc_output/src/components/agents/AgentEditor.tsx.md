# AgentEditor.tsx — Agent编辑器组件

> **一句话总结**：提供Agent配置的交互式编辑功能，支持修改工具、模型和颜色。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/AgentEditor.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 编辑现有Agent的配置 |

---

## 功能概述

`AgentEditor`组件允许用户编辑已存在的Agent配置。它提供一个菜单驱动的界面，支持以下编辑操作：
- 在外部编辑器中打开Agent定义文件
- 编辑Agent可用的工具集
- 修改Agent使用的AI模型
- 更改Agent的颜色标识

编辑完成后，变更会保存到Agent的定义文件中，并更新应用状态。

---

## 核心内容详解

### 主要类型

- **EditMode**: 编辑模式类型，包括`'menu'`、`'edit-tools'`、`'edit-color'`、`'edit-model'`
- **SaveChanges**: 保存变更的数据结构，包含可选的工具、颜色、模型字段

### 主要函数

- **AgentEditor**: 主组件
  - **参数**: `Props`，包含`agent`（要编辑的Agent）、`tools`（可用工具）、`onSaved`（保存回调）、`onBack`（返回回调）
  - **状态管理**: 
    - `editMode`: 当前编辑模式
    - `selectedMenuIndex`: 菜单选中项
    - `error`: 错误信息
    - `selectedColor`: 选中的颜色

- **handleOpenInEditor**: 在外部编辑器中打开Agent文件
- **handleSave**: 保存变更到文件和状态
  - 验证是否有变更
  - 调用`updateAgentFile`写入文件
  - 更新全局App状态
  - 调用`onSaved`回调

### 菜单项

1. **Open in editor** - 在外部编辑器中打开
2. **Edit tools** - 进入工具选择器
3. **Edit model** - 进入模型选择器
4. **Edit color** - 进入颜色选择器

---

## 设计要点

- 使用菜单驱动的交互模式，适合终端环境
- 子编辑器（ToolSelector/ModelSelector/ColorPicker）通过`editMode`状态切换
- 保存时同步更新文件系统和应用状态
- 支持错误处理和用户反馈

---

## 与其他文件的关系

- **依赖**:
  - `ToolSelector`, `ModelSelector`, `ColorPicker` - 子编辑器组件
  - `updateAgentFile` - 文件更新
  - `useSetAppState` - 全局状态更新
- **被依赖**:
  - `AgentsMenu` - 作为编辑视图使用

---

## 注意事项

- 只有自定义Agent和插件Agent可以编辑，内置Agent不可编辑
- 编辑工具需要重新选择整个工具集
- 颜色变更会立即应用到视觉标识
