# select-option.tsx

选择选项的基础渲染组件，负责渲染单个选项的 UI。

## 基本信息

- **文件路径**: `src/components/CustomSelect/select-option.tsx`
- **组件名称**: SelectOption
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

SelectOption 是自定义选择组件的基础选项渲染组件。它封装了 ListItem 组件，提供统一的选项样式，包括聚焦状态、选中状态、描述信息和滚动箭头指示器。

## 核心内容

### Props 接口 (SelectOptionProps)

| 属性 | 类型 | 说明 |
|------|------|------|
| isFocused | boolean | 选项是否处于聚焦状态 |
| isSelected | boolean | 选项是否被选中 |
| children | ReactNode | 选项内容（标签） |
| description | string | 选项描述（显示在标签下方） |
| shouldShowDownArrow | boolean | 是否显示向下滚动箭头 |
| shouldShowUpArrow | boolean | 是否显示向上滚动箭头 |
| declareCursor | boolean | 是否声明终端光标位置（默认为 true，子组件声明光标时设为 false） |

### 功能特性

1. **状态显示**:
   - 聚焦状态：高亮显示当前聚焦的选项
   - 选中状态：显示选项的选中标记

2. **滚动指示器**:
   - 向上箭头：表示上方有更多选项
   - 向下箭头：表示下方有更多选项

3. **描述显示**: 支持在选项下方显示描述文字

4. **光标管理**: 可通过 `declareCursor` 控制是否声明终端光标位置

## 设计要点

- 使用 React Compiler 进行编译优化
- 基于 `ListItem` 组件构建，复用其样式和布局
- `styled={false}` 表示不使用默认样式，由父组件控制
- 支持通过 children 传递复杂的标签内容

## 与其他文件的关系

- **被导入**:
  - `select.tsx` - 主选择组件使用 SelectOption 渲染选项
  - `SelectMulti.tsx` - 多选组件使用 SelectOption 渲染选项
  - `select-input-option.tsx` - 输入选项组件也使用 SelectOption

- **导入依赖**:
  - `../design-system/ListItem.js` - 列表项基础组件

## 备注

- 该组件是无状态展示组件，所有状态由父组件传递
- `declareCursor` 属性用于处理嵌套输入框的场景（如 BaseTextInput 自己声明光标）
