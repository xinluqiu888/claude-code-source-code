# useFeedbackSurvey.tsx

反馈调查 Hook，管理反馈调查的完整生命周期。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/useFeedbackSurvey.tsx`
- **导出函数**: useFeedbackSurvey
- **类型**: React Custom Hook

## 功能概述

useFeedbackSurvey 管理反馈调查的完整流程，包括调查触发、响应处理、分析事件记录、转录分享等功能。它根据消息历史、加载状态等条件决定是否显示调查。

## 核心内容

### Props 接口

| 属性 | 类型 | 说明 |
|------|------|------|
| messages | Message[] | 消息历史 |
| isLoading | boolean | 是否正在加载 |
| hasActivePrompt | boolean | 是否有活动提示（可选，默认 false） |
| options | { enabled?: boolean } | 选项（可选，默认 { enabled: true }） |

### 返回值

| 属性 | 类型 | 说明 |
|------|------|------|
| state | SurveyState | 当前状态 |
| lastResponse | FeedbackSurveyResponse \| null | 上次响应 |
| handleSelect | (selected: FeedbackSurveyResponse) => boolean | 处理选择 |
| handleTranscriptSelect | (selected: TranscriptShareResponse) => void | 处理转录选择 |

### 主要常量

| 常量 | 值 | 说明 |
|------|-----|------|
| HIDE_THANKS_AFTER_MS | 3000 | 感谢显示时间（3秒） |
| FEEDBACK_SURVEY_GATE | 'tengu_feedback_survey_gate' | 功能开关 |
| SURVEY_PROBABILITY | 0.1 | 调查概率（10%） |
| TRANSCRIPT_SHARE_TRIGGER | 'bad_feedback_survey' \| 'good_feedback_survey' | 转录分享触发器 |

### 调查触发条件

调查在以下所有条件满足时触发：
1. 功能开关已启用（GrowthBook）
2. 不在加载中
3. 没有活动提示
4. 策略允许产品反馈
5. 环境变量未禁用调查
6. 达到消息阈值（每N条消息）
7. 随机概率检查通过（10%）

### 分析事件

| 事件 | 触发时机 |
|------|----------|
| tengu_feedback_survey_event (appeared) | 调查显示 |
| tengu_feedback_survey_event (responded) | 用户响应 |
| tengu_feedback_survey_event (transcript_prompt_appeared) | 转录提示显示 |
| tengu_feedback_survey_event (transcript_share_*) | 转录分享选择 |
| tengu_feedback_survey_event (transcript_share_submitted/failed) | 转录提交结果 |

## 设计要点

- 使用 `useSurveyState` 管理状态机
- 使用 `useRef` 跟踪已评估的消息，避免重复触发
- 支持转录分享功能（仅在 bad/good 响应后）
- 集成 GrowthBook 功能开关

## 与其他文件的关系

- **导入依赖**:
  - `./useSurveyState.js` - 调查状态 hook
  - `./submitTranscriptShare.js` - 转录分享提交
  - `src/services/analytics/index.js` - 分析服务
  - `src/services/policyLimits/index.js` - 策略限制

## 备注

- 消息阈值和概率控制调查频率，避免过度打扰用户
- 转录分享需要额外的用户确认
- 分析事件用于追踪调查参与度和转录分享率
