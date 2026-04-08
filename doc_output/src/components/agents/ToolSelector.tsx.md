# ToolSelector.tsx — Agent工具选择器

> **一句话总结**：提供工具集配置界面，按类别组织和管理Agent可用工具。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/ToolSelector.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 配置Agent可用的工具集合 |

---

## 功能概述

`ToolSelector`组件允许用户为Agent配置可用的工具集。它提供：
- 按类别组织的工具选择（只读、编辑、执行、MCP、其他）
- 批量选择/取消选择某一类工具
- 全选/取消全选功能
- MCP工具按服务器分组显示
- 键盘导航和操作确认

---

## 核心内容详解

### 主要类型

- **Props**:
  - `tools`: 所有可用工具
  - `initialTools`: 初始选中的工具列表
  - `onComplete`: 完成回调
  - `onCancel?`: 取消回调

- **ToolBucket**: 工具类别
  - `name`: 类别名称
  - `toolNames`: 工具名称集合
  - `isMcp?`: 是否为MCP类别

- **ToolBuckets**: 工具类别集合
  - READ_ONLY, EDIT, EXECUTION, MCP, OTHER

### 主要函数

- **getToolBuckets**: 获取预定义的工具类别
  - READ_ONLY: GlobTool, GrepTool, FileReadTool等
  - EDIT: FileEditTool, FileWriteTool, NotebookEditTool
  - EXECUTION: BashTool, TungstenTool
  - MCP: 动态MCP工具
  - OTHER: 其他未分类工具

- **getMcpServerBuckets**: 按MCP服务器分组获取工具

- **ToolSelector**: 主组件函数
  - **状态管理**:
    - `selectedTools`: 已选中的工具列表
    - `focusIndex`: 当前聚焦的项
    - `showIndividualTools`: 是否显示单个工具

- **handleToggleTool**: 切换单个工具的选中状态
- **handleToggleTools**: 批量切换多个工具的选中状态
- **handleSave**: 保存选择，过滤掉无效的工具名

### 渲染逻辑

1. **工具分类显示**: 按READ_ONLY、EDIT、EXECUTION、MCP等类别显示
2. **MCP工具**: 按服务器名称分组显示
3. **单个工具列表**: 可展开显示具体工具
4. **操作按钮**: Save、Cancel、Toggle individual tools

---

## 设计要点

- 使用React Compiler优化渲染性能
- 工具分类便于用户理解功能范围
- 支持通配符(*)选择所有工具
- 过滤掉内置Agent工具和异步工具

---

## 与其他文件的关系

- **依赖**:
  - `filterToolsForAgent` - 过滤Agent可用工具
  - 各类Tool定义 - 构建工具类别
  - `useKeybinding` - 键盘快捷键
- **被依赖**:
  - `AgentEditor` - 作为工具编辑子组件

---

## 注意事项

- 只显示适合自定义Agent的工具（排除内置工具和异步工具）
- MCP工具动态获取，按服务器分组
- 保存时会验证工具名称的有效性
- 支持选择"*"表示所有工具
