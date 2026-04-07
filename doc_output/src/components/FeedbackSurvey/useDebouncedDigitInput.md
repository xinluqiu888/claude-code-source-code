# useDebouncedDigitInput.ts

防抖数字输入 Hook，用于处理调查数字输入。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/useDebouncedDigitInput.ts`
- **导出函数**: useDebouncedDigitInput
- **类型**: React Custom Hook

## 功能概述

useDebouncedDigitInput 用于检测用户在提示输入框中输入单个有效数字，并在短暂延迟后触发回调。这可以防止意外提交（例如用户在输入编号列表时）。

## 核心内容

### Props 接口

| 属性 | 类型 | 说明 |
|------|------|------|
| inputValue | string | 当前输入值 |
| setInputValue | (value: string) => void | 设置输入值回调 |
| isValidDigit | (char: string) => char is T | 验证数字是否有效 |
| onDigit | (digit: T) => void | 数字确认回调 |
| enabled | boolean | 是否启用，默认 true |
| once | boolean | 是否只触发一次，默认 false |
| debounceMs | number | 防抖延迟（毫秒），默认 400ms |

### 默认防抖延迟

```typescript
const DEFAULT_DEBOUNCE_MS = 400
```

### 工作原理

1. 监听 inputValue 变化
2. 如果最后一个字符是有效数字，启动防抖定时器
3. 在延迟期间如果有新输入，取消之前的定时器
4. 延迟结束后，从输入中移除该数字并触发 onDigit 回调

## 设计要点

- 使用 `normalizeFullWidthDigits` 处理全角数字
- 使用 ref 存储回调，避免每次渲染重新订阅
- 支持一次性触发模式（once: true）
- 清理机制确保组件卸载时清除定时器

## 与其他文件的关系

- **被导入**:
  - `FeedbackSurveyView.tsx` - 反馈调查视图
  - `TranscriptSharePrompt.tsx` - 转录分享提示
  - `FeedbackSurvey.tsx` - 主调查组件（Thanks 视图）

- **导入依赖**:
  - `../../utils/stringUtils.js` - 字符串工具（全角数字规范化）

## 备注

- 400ms 的延迟既能保证有意输入的即时感，又能避免因输入编号列表（如 "1. First item"）导致的意外提交
- 触发后会自动从输入中移除数字字符
