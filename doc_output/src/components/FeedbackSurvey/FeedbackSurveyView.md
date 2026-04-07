# FeedbackSurveyView.tsx

反馈调查视图组件，显示调查问题和选项。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/FeedbackSurveyView.tsx`
- **组件名称**: FeedbackSurveyView
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

FeedbackSurveyView 显示反馈调查的 UI，包括问题提示和数字选项（1-3: Bad/Fine/Good，0: Dismiss）。它使用防抖数字输入来处理用户响应。

## 核心内容

### Props 接口

| 属性 | 类型 | 说明 |
|------|------|------|
| onSelect | (option: FeedbackSurveyResponse) => void | 选择回调 |
| inputValue | string | 当前输入值 |
| setInputValue | (value: string) => void | 设置输入值回调 |
| message | string | 自定义消息（可选，默认：How is Claude doing this session?） |

### 响应选项映射

| 输入 | 响应值 |
|------|--------|
| '0' | 'dismissed' |
| '1' | 'bad' |
| '2' | 'fine' |
| '3' | 'good' |

### 导出函数

- **isValidResponseInput(input: string)**: 验证输入是否为有效响应（'0'-'3'）

## 设计要点

- 使用 `useDebouncedDigitInput` 处理数字输入
- 默认消息可自定义，用于不同场景的调查
- 使用青色（ansi:cyan）高亮显示选项数字

## 与其他文件的关系

- **导入依赖**:
  - `./useDebouncedDigitInput.js` - 防抖数字输入 hook
  - `../../ink.js` - Ink UI 组件

## 备注

- 该组件是无状态展示组件，状态由父组件管理
- 使用数字键（0-3）快速响应，无需 Enter 确认
