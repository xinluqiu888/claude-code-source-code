# usePostCompactSurvey.tsx

会话压缩后调查 Hook，在内存压缩后触发反馈调查。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/usePostCompactSurvey.tsx`
- **导出函数**: usePostCompactSurvey
- **类型**: React 函数组件 (使用 React Compiler)

## 功能概述

usePostCompactSurvey 在会话内存压缩完成后触发反馈调查。它监控消息历史中的压缩边界消息，在新的消息到达后显示调查。

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
| handleSelect | (selected: FeedbackSurveyResponse) => void | 处理选择 |

### 主要常量

| 常量 | 值 | 说明 |
|------|-----|------|
| HIDE_THANKS_AFTER_MS | 3000 | 感谢显示时间 |
| POST_COMPACT_SURVEY_GATE | 'tengu_post_compact_survey' | 功能开关 |
| SURVEY_PROBABILITY | 0.2 | 调查概率（20%） |

### hasMessageAfterBoundary 函数

检查在指定的压缩边界 UUID 之后是否有新的用户或 assistant 消息：

```typescript
function hasMessageAfterBoundary(
  messages: Message[],
  boundaryUuid: string,
): boolean
```

### 调查触发流程

1. 使用 useMemo 提取所有压缩边界消息的 UUID
2. 检测新出现的边界（不在 seenCompactBoundaries 中）
3. 将新边界标记为 pending（等待下一条消息）
4. 当下一条消息到达时，触发调查（20% 概率）

### 分析事件

| 事件 | 属性 |
|------|------|
| tengu_post_compact_survey_event (appeared) | appearance_id, session_memory_compaction_enabled |
| tengu_post_compact_survey_event (responded) | appearance_id, response, session_memory_compaction_enabled |

## 设计要点

- 使用 React Compiler 优化渲染
- 延迟显示：在压缩边界出现后，等待下一条消息才显示调查
- 使用 ref 跟踪已见过的边界和待处理的边界
- 集成 session memory compaction 状态追踪

## 与其他文件的关系

- **导入依赖**:
  - `./useSurveyState.js` - 调查状态 hook
  - `../../services/compact/sessionMemoryCompact.js` - 会话内存压缩
  - `../../utils/messages.js` - `isCompactBoundaryMessage` 检测

## 备注

- 调查在新消息到达后才显示，避免打扰正在进行的对话
- 20% 的概率平衡了数据收集需求和用户体验
- 分析事件包含会话内存压缩启用状态，用于功能效果评估
