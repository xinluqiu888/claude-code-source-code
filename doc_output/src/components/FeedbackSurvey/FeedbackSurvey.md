# FeedbackSurvey.tsx

反馈调查主组件，根据当前状态显示不同的调查视图。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/FeedbackSurvey.tsx`
- **组件名称**: FeedbackSurvey
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

FeedbackSurvey 是反馈调查系统的主组件，根据状态机管理不同的显示状态（closed、open、thanks、transcript_prompt、submitting、submitted），并根据当前状态渲染相应的视图。

## 核心内容

### Props 接口

| 属性 | 类型 | 说明 |
|------|------|------|
| state | SurveyState | 当前状态：closed、open、thanks、transcript_prompt、submitting、submitted |
| lastResponse | FeedbackSurveyResponse \| null | 上次响应结果 |
| handleSelect | (selected: FeedbackSurveyResponse) => void | 选择回调 |
| handleTranscriptSelect | (selected: TranscriptShareResponse) => void | 转录分享选择回调 |
| inputValue | string | 当前输入值 |
| setInputValue | (value: string) => void | 设置输入值回调 |
| onRequestFeedback | () => void | 请求反馈回调（可选） |
| message | string | 自定义消息（可选） |

### 状态渲染逻辑

| 状态 | 渲染内容 |
|------|----------|
| closed | null（不显示） |
| thanks | FeedbackSurveyThanks 组件 |
| submitted | 成功消息（✓ Thanks for sharing your transcript!） |
| submitting | 加载提示（Sharing transcript...） |
| transcript_prompt | TranscriptSharePrompt 组件（仅当用户选择 bad/good 时） |
| open | FeedbackSurveyView 组件 |

### 子组件

#### FeedbackSurveyThanks

感谢视图组件，显示反馈感谢信息。

**特殊功能**:
- 如果用户选择 "good" 且提供 onRequestFeedback 回调，显示 "Press [1] to tell us what went well" 提示
- 使用 `useDebouncedDigitInput` 监听 "1" 键触发反馈请求
- 根据最后响应显示不同提示信息

## 设计要点

- 使用 React Compiler 优化渲染
- 状态驱动渲染，每个状态对应不同 UI
- 转录分享提示仅在用户选择 bad/good 时显示
- 使用 `isValidResponseInput` 过滤无效输入

## 与其他文件的关系

- **导入依赖**:
  - `./FeedbackSurveyView.js` - 调查视图组件
  - `./TranscriptSharePrompt.js` - 转录分享提示组件
  - `./useDebouncedDigitInput.js` - 防抖数字输入 hook
  - `src/services/analytics/index.js` - 分析事件记录

## 备注

- 输入验证：当用户输入非响应字符时隐藏调查，防止意外提交
- 转录分享：仅在输入值为 "1"、"2"、"3" 时显示提示
