# select-input-option.tsx

支持文本输入的选择选项组件，允许用户在选项中进行文本输入。

## 基本信息

- **文件路径**: `src/components/CustomSelect/select-input-option.tsx`
- **组件名称**: SelectInputOption
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

SelectInputOption 是一个特殊的选择选项组件，支持在选项内进行文本输入。它结合了 SelectOption 的列表展示功能和 TextInput 的输入功能，常用于需要提供输入反馈的选择场景（如"是，并允许..."这类选项）。

## 核心内容

### Props 接口 (Props<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| option | OptionWithDescription<T> (type: 'input') | 输入类型选项配置 |
| isFocused | boolean | 是否聚焦 |
| isSelected | boolean | 是否选中 |
| shouldShowDownArrow | boolean | 是否显示向下箭头 |
| shouldShowUpArrow | boolean | 是否显示向上箭头 |
| maxIndexWidth | number | 序号最大宽度 |
| index | number | 选项序号 |
| inputValue | string | 当前输入值 |
| onInputChange | (value: string) => void | 输入变更回调 |
| onSubmit | (value: string) => void | 提交回调 |
| onExit | () => void | 退出回调 |
| layout | 'compact' \| 'expanded' | 布局模式 |
| children | ReactNode | 子元素（如选中标记） |
| showLabel | boolean | 是否显示标签 |
| onOpenEditor | (...) => void | 打开外部编辑器回调 |
| resetCursorOnUpdate | boolean | 更新时重置光标 |
| onImagePaste | (...) => void | 图片粘贴回调 |
| pastedContents | Record<number, PastedContent> | 粘贴的内容 |
| onRemoveImage | (id: number) => void | 移除图片回调 |
| imagesSelected | boolean | 图片是否选中 |
| selectedImageIndex | number | 选中的图片索引 |
| onImagesSelectedChange | (selected: boolean) => void | 图片选中状态变更回调 |
| onSelectedImageIndexChange | (index: number) => void | 选中图片索引变更回调 |

### 主要功能

1. **文本输入**: 内置 TextInput 组件，支持在选项内输入文本
2. **光标管理**: 支持 `resetCursorOnUpdate` 在聚焦或值更新时重置光标位置
3. **外部编辑器**: 支持 Ctrl+G 快捷键打开外部编辑器
4. **图片粘贴**: 支持从剪贴板粘贴图片
5. **图片附件管理**: 支持显示、选择和删除图片附件

### 状态管理

- **cursorOffset**: 光标位置偏移
- **isUserEditing**: 是否用户正在编辑（用于防止自动光标重置）
- **imageAttachments**: 图片附件列表
- **showLabel**: 是否显示标签（受 option.showLabelWithValue 影响）

## 设计要点

- 使用 `useKeybinding` 注册 chat:externalEditor 快捷键
- 使用 `useKeybindings` 处理图片附件导航快捷键
- 通过 `useEffect` 监听输入值变化，按需重置光标
- 使用 `ClickableImageRef` 显示可点击的图片引用
- 使用 `ConfigurableShortcutHint` 显示快捷键提示

## 与其他文件的关系

- **被导入**:
  - `select.tsx` - 主选择组件使用 SelectInputOption 渲染输入选项
  - `SelectMulti.tsx` - 多选组件使用 SelectInputOption 渲染输入选项

- **导入依赖**:
  - `./select-option.js` - 基础选项组件
  - `../TextInput.js` - 文本输入组件
  - `../ClickableImageRef.js` - 可点击图片引用组件
  - `../ConfigurableShortcutHint.js` - 快捷键提示组件
  - `../design-system/Byline.js` - 副标题组件
  - `../../keybindings/useKeybinding.js` - 快捷键 hook
  - `../../utils/imagePaste.js` - 图片粘贴工具

## 备注

- 图片粘贴使用 `getImageFromClipboard()` 从剪贴板获取图片数据
- 支持在图片选择模式下使用方向键导航图片
- 使用 Ctrl+N/P 或 J/K 进行上下导航（在输入模式下）
