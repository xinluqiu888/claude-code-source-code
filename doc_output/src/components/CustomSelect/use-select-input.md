# use-select-input.ts

选择组件的输入处理 Hook，处理键盘输入和快捷键。

## 基本信息

- **文件路径**: `src/components/CustomSelect/use-select-input.ts`
- **导出函数**: useSelectInput
- **类型**: React Custom Hook

## 功能概述

useSelectInput 负责处理选择组件的所有键盘输入，包括导航键（上下、分页）、选择键（Enter、Space、数字键）、取消键（Escape）等。它还处理输入模式和图片选择模式的特殊逻辑。

## 核心内容

### Props 接口 (UseSelectProps<T>)

| 属性 | 类型 | 说明 |
|------|------|------|
| isDisabled | boolean | 是否禁用，默认 false |
| disableSelection | boolean \| 'numeric' | 禁用选择模式 |
| state | SelectState<T> | 选择状态（必需） |
| options | OptionWithDescription<T>[] | 选项列表（必需） |
| isMultiSelect | boolean | 是否多选，默认 false |
| onUpFromFirstItem | () => void | 从第一个选项向上回调 |
| onDownFromLastItem | () => void | 从最后一个选项向下回调 |
| onInputModeToggle | (value: T) => void | 输入模式切换回调 |
| inputValues | Map<T, string> | 输入选项的当前值 |
| imagesSelected | boolean | 图片选择模式是否激活 |
| onEnterImageSelection | () => boolean | 进入图片选择模式回调 |

### 处理逻辑

1. **注册为 Overlay**:
   - 使用 `useRegisterOverlay` 注册选择组件为覆盖层
   - 确保 Escape 键被正确处理而不被其他处理器拦截

2. **键盘快捷键绑定**:
   - `select:next` - 下一个选项
   - `select:previous` - 上一个选项
   - `select:accept` - 确认选择
   - `select:cancel` - 取消

3. **原始输入处理** (useInput):
   - **Tab**: 切换输入模式
   - **PageUp/PageDown**: 分页导航
   - **Space**: 多选切换 / 输入空格
   - **数字键**: 直接选择对应索引的选项
   - **输入模式下的导航**: Ctrl+N/P、上下箭头

4. **数字键处理逻辑**:
   - 输入类型选项且有值：自动提交
   - 输入类型选项且 allowEmptySubmitToCancel：提交
   - 输入类型选项且为空：聚焦该选项
   - 普通选项：直接选择

5. **输入模式特殊处理**:
   - 输入模式下禁止导航快捷键（j/k/Enter）
   - 图片选择模式下抑制所有输入，由 Attachments 快捷键处理
   - DOWN 箭头可进入图片选择模式

## 设计要点

- 区分 keybindings（快捷键）和 useInput（原始输入）
- 输入模式下排除导航快捷键，让输入框接收字符
- 支持全角数字和空格的规范化处理
- 使用 isInInput 判断当前是否在输入模式

## 与其他文件的关系

- **被导入**:
  - `select.tsx` - 主选择组件使用该 hook 处理输入

- **导入依赖**:
  - `../../context/overlayContext.js` - 覆盖层上下文
  - `../../keybindings/useKeybinding.js` - 快捷键 hook
  - `../../utils/stringUtils.js` - 字符串规范化工具

## 备注

- 快捷键在输入模式下被排除，避免与输入框冲突
- 图片选择模式下需要特殊处理，防止与 Attachments 快捷键冲突
- 数字键选择时自动处理输入类型选项的特殊逻辑
