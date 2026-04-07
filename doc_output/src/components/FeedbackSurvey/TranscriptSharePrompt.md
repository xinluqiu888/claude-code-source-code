# TranscriptSharePrompt.tsx

转录分享提示组件，询问用户是否允许查看会话转录。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/TranscriptSharePrompt.tsx`
- **组件名称**: TranscriptSharePrompt
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

TranscriptSharePrompt 在用户完成反馈调查后显示，询问是否允许 Anthropic 查看会话转录以帮助改进 Claude Code。提供三个选项：同意分享、拒绝、不再询问。

## 核心内容

### Props 接口

| 属性 | 类型 | 说明 |
|------|------|------|
| onSelect | (option: TranscriptShareResponse) => void | 选择回调 |
| inputValue | string | 当前输入值 |
| setInputValue | (value: string) => void | 设置输入值回调 |

### TranscriptShareResponse 类型

```typescript
type TranscriptShareResponse = 'yes' | 'no' | 'dont_ask_again'
```

### 响应选项映射

| 输入 | 响应值 |
|------|--------|
| '1' | 'yes' |
| '2' | 'no' |
| '3' | 'dont_ask_again' |

## 设计要点

- 使用 `BLACK_CIRCLE` 符号作为标题前缀
- 显示隐私政策链接供用户了解详情
- 使用 `useDebouncedDigitInput` 处理数字输入
- 选项使用青色（ansi:cyan）高亮

## 与其他文件的关系

- **导入依赖**:
  - `../../constants/figures.js` - 符号常量
  - `../../ink.js` - Ink UI 组件
  - `./useDebouncedDigitInput.js` - 防抖数字输入 hook

## 备注

- 该组件在反馈调查流程的 transcript_prompt 阶段显示
- "Don't ask again" 选项会持久化到全局配置中
