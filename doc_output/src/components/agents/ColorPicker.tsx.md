# ColorPicker.tsx — Agent颜色选择器

> **一句话总结**：交互式颜色选择组件，支持为Agent设置颜色标识。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/ColorPicker.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 提供Agent颜色选择和预览功能 |

---

## 功能概述

`ColorPicker`组件允许用户为Agent选择颜色标识。它提供：
- 预定义颜色列表选择
- "Automatic"自动颜色选项
- 实时预览选中的颜色效果
- 键盘导航支持（上下箭头选择，回车确认）

---

## 核心内容详解

### 主要类型

- **ColorOption**: 颜色选项类型，`AgentColorName | 'automatic'`
- **Props**:
  - `agentName`: Agent名称（用于预览）
  - `currentColor?`: 当前选中的颜色
  - `onConfirm`: 确认选择回调

### 常量

- **COLOR_OPTIONS**: 所有可用颜色选项数组，包含'automatic'和所有预定义颜色

### 主要函数

- **ColorPicker**: 主组件函数
  - **状态管理**:
    - `selectedIndex`: 当前选中项的索引
  - **初始化**: 根据currentColor设置默认选中项

- **handleKeyDown**: 键盘事件处理
  - `up`: 向上移动选择
  - `down`: 向下移动选择
  - `return`: 确认选择，如为'automatic'则返回undefined

### 渲染逻辑

1. **颜色选项列表**: 显示所有可用颜色选项
   - 使用pointer图标标识选中项
   - 显示颜色方块和名称
   - "Automatic color"选项无颜色方块

2. **预览区域**: 显示Agent名称应用选中颜色后的效果
   - 自动模式使用inverse样式
   - 具体颜色使用backgroundColor设置

---

## 设计要点

- 使用React Compiler优化渲染
- 循环导航（到顶部后按上键到底部，反之亦然）
- 实时预览让用户直观看到效果
- 自动选项返回undefined，表示由系统决定颜色

---

## 与其他文件的关系

- **依赖**:
  - `AGENT_COLOR_TO_THEME_COLOR`, `AGENT_COLORS` - 颜色定义
  - `figures` - 指针图标
  - `capitalize` - 字符串格式化
- **被依赖**:
  - `AgentEditor` - 作为颜色编辑子组件

---

## 注意事项

- 选中'automatic'时返回undefined而非字符串'automatic'
- 颜色名称首字母会自动大写显示
- 预览区域使用`@{agentName}`格式显示
