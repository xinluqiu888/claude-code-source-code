# select.tsx

自定义选择组件的主实现文件，支持单选、文本输入选项、图片粘贴等高级功能。

## 基本信息

- **文件路径**: `src/components/CustomSelect/select.tsx`
- **组件名称**: Select
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

Select 组件是一个功能丰富的终端选择器，支持键盘导航、文本输入、图片粘贴、分页滚动等交互。它是 Claude Code 中各种菜单和选择对话框的基础组件。

## 核心内容

### Props 接口 (SelectProps<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| isDisabled | boolean | 是否禁用输入，默认 false |
| disableSelection | boolean \| 'numeric' | 禁用选择但允许滚动 |
| hideIndexes | boolean | 隐藏选项序号，默认 false |
| visibleOptionCount | number | 可见选项数量，默认 5 |
| highlightText | string | 高亮显示的文本 |
| options | OptionWithDescription<T>[] | 选项列表 |
| defaultValue | T | 默认值 |
| onCancel | () => void | 取消回调 |
| onChange | (value: T) => void | 选择变更回调 |
| onFocus | (value: T) => void | 聚焦变更回调 |
| defaultFocusValue | T | 初始聚焦值 |
| layout | 'compact' \| 'expanded' \| 'compact-vertical' | 布局模式 |
| inlineDescriptions | boolean | 内联显示描述 |
| onUpFromFirstItem | () => void | 从第一个选项向上回调 |
| onDownFromLastItem | () => void | 从最后一个选项向下回调 |
| onInputModeToggle | (value: T) => void | 输入模式切换回调 |
| onOpenEditor | (currentValue: string, setValue) => void | 打开外部编辑器回调 |
| onImagePaste | (...) => void | 图片粘贴回调 |
| pastedContents | Record<number, PastedContent> | 粘贴的内容 |
| onRemoveImage | (id: number) => void | 移除图片回调 |

### Option 类型 (OptionWithDescription<T>)

支持两种选项类型：

1. **文本选项** (`type: 'text'`):
   - label: ReactNode - 选项标签
   - value: T - 选项值
   - description?: string - 描述
   - disabled?: boolean - 是否禁用

2. **输入选项** (`type: 'input'`):
   - 包含文本选项的所有字段
   - onChange: (value: string) => void - 输入变更回调
   - placeholder?: string - 占位符
   - initialValue?: string - 初始值
   - allowEmptySubmitToCancel?: boolean - 允许空提交取消
   - showLabelWithValue?: boolean - 始终显示标签
   - labelValueSeparator?: string - 标签值分隔符
   - resetCursorOnUpdate?: boolean - 更新时重置光标

### 辅助函数

- **getTextContent(node: ReactNode)**: 从 ReactNode 提取文本内容用于宽度计算

## 设计要点

- 使用 React Compiler 优化渲染性能
- 支持 `figures` 图标库显示选中标记
- 通过 `useSelectState` 管理选择状态
- 通过 `useSelectInput` 处理键盘输入
- 使用 `useDeclaredCursor` 管理终端光标
- 支持全角数字和空格的规范化处理

## 与其他文件的关系

- **被导入**:
  - `PluginHintMenu.tsx` - 插件提示菜单使用 Select 组件
  - 各种需要选择功能的组件

- **导入依赖**:
  - `./use-select-state.js` - 选择状态管理 hook
  - `./use-select-input.js` - 输入处理 hook
  - `./select-option.js` - 选项子组件
  - `./select-input-option.js` - 输入选项子组件
  - `../TextInput.js` - 文本输入组件
  - `../ink.js` - Ink UI 库
  - `figures` - 终端图标

## 备注

- 组件支持三种布局模式：compact（紧凑）、expanded（展开）、compact-vertical（垂直紧凑）
- 输入选项支持 Ctrl+G 快捷键打开外部编辑器
- 支持图片粘贴功能，需要配合 `onImagePaste` 回调使用
- 选项变更时会自动触发 onFocus 回调（如果提供）
