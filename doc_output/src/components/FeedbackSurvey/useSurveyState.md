# useSurveyState.tsx

调查状态管理 Hook，提供调查状态机的完整管理。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/useSurveyState.tsx`
- **导出函数**: useSurveyState
- **类型**: React Custom Hook

## 功能概述

useSurveyState 提供了一个完整的状态机来管理调查流程，包括打开调查、处理响应、显示感谢、转录分享提示等状态转换。

## 核心内容

### SurveyState 类型

```typescript
type SurveyState = 
  | 'closed'      // 关闭状态
  | 'open'        // 打开状态（显示调查）
  | 'thanks'      // 感谢状态
  | 'transcript_prompt'  // 转录分享提示
  | 'submitting'  // 提交中
  | 'submitted'   // 已提交
```

### Props 接口 (UseSurveyStateOptions)

| 属性 | 类型 | 说明 |
|------|------|------|
| hideThanksAfterMs | number | 感谢显示持续时间（毫秒） |
| onOpen | (appearanceId: string) => void \| Promise<void> | 调查打开回调 |
| onSelect | (appearanceId: string, selected: FeedbackSurveyResponse) => void \| Promise<void> | 选择回调 |
| shouldShowTranscriptPrompt | (selected: FeedbackSurveyResponse) => boolean | 是否显示转录分享提示（可选） |
| onTranscriptPromptShown | (appearanceId: string, surveyResponse: FeedbackSurveyResponse) => void | 转录提示显示回调（可选） |
| onTranscriptSelect | (appearanceId: string, selected: TranscriptShareResponse, surveyResponse: FeedbackSurveyResponse \| null) => boolean \| Promise<boolean> | 转录选择回调（可选） |

### 返回值

| 属性 | 类型 | 说明 |
|------|------|------|
| state | SurveyState | 当前状态 |
| lastResponse | FeedbackSurveyResponse \| null | 上次响应 |
| open | () => void | 打开调查方法 |
| handleSelect | (selected: FeedbackSurveyResponse) => boolean | 处理选择方法 |
| handleTranscriptSelect | (selected: TranscriptShareResponse) => void | 处理转录选择方法 |

### 状态转换逻辑

1. **closed -> open**: 调用 open() 方法
2. **open -> thanks**: 选择非 'dismissed' 响应且不显示转录提示
3. **open -> closed**: 选择 'dismissed'
4. **open -> transcript_prompt**: 选择后 shouldShowTranscriptPrompt 返回 true
5. **transcript_prompt -> submitting/closed**: 根据转录选择结果
6. **thanks/submitted -> closed**: 延迟后自动关闭

## 设计要点

- 使用 `randomUUID` 生成唯一 appearanceId 用于追踪
- 使用 ref 存储 lastResponse 以便在异步回调中访问
- 自动关闭逻辑：感谢/提交状态在指定时间后自动关闭
- 支持转录分享流程的完整状态管理

## 与其他文件的关系

- **被导入**:
  - `useFeedbackSurvey.tsx` - 反馈调查 hook
  - `useMemorySurvey.tsx` - 内存调查 hook
  - `usePostCompactSurvey.tsx` - 压缩后调查 hook

- **导入依赖**:
  - `crypto` 模块的 `randomUUID`

## 备注

- appearanceId 用于关联分析事件和调查实例
- handleSelect 返回 boolean 表示是否进入转录提示状态
- 自动关闭使用 setTimeout 实现
