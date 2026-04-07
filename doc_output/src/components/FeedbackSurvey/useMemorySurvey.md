# useMemorySurvey.tsx

内存调查 Hook，在检测到内存相关操作后触发反馈调查。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/useMemorySurvey.tsx`
- **导出函数**: useMemorySurvey
- **类型**: React Custom Hook

## 功能概述

useMemorySurvey 专门用于在 Claude 读取内存文件后触发反馈调查。它检测消息历史中是否包含内存文件读取操作，并在满足条件时以一定概率显示调查。

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
| handleTranscriptSelect | (selected: TranscriptShareResponse) => void | 处理转录选择 |

### 主要常量

| 常量 | 值 | 说明 |
|------|-----|------|
| HIDE_THANKS_AFTER_MS | 3000 | 感谢显示时间 |
| MEMORY_SURVEY_GATE | 'tengu_dunwich_bell' | 功能开关 |
| SURVEY_PROBABILITY | 0.2 | 调查概率（20%） |
| TRANSCRIPT_SHARE_TRIGGER | 'memory_survey' | 转录分享触发器标识 |
| MEMORY_WORD_RE | /\bmemor(?:y\|ies)\b/i | 内存关键词正则 |

### hasMemoryFileRead 函数

检测消息历史中是否包含自动管理的内存文件读取操作：
1. 遍历所有 assistant 类型的消息
2. 检查消息内容中的 tool_use 块
3. 查找 FileReadTool 调用
4. 验证文件路径是否为自动管理的内存文件

### 调查触发条件

1. 功能开关已启用（GrowthBook）
2. 自动内存功能已启用
3. 反馈调查未被禁用
4. 策略允许产品反馈
5. 环境变量未禁用调查
6. 最后一条 assistant 消息包含 "memory" 关键词
7. 检测到内存文件读取操作
8. 随机概率检查通过（20%）

## 设计要点

- 使用 ref 跟踪已评估的 assistant UUID，避免重复触发
- 使用 ref 缓存内存读取状态，优化性能
- `/clear` 命令会重置 ref，确保新会话不受旧会话影响
- 支持转录分享流程

## 与其他文件的关系

- **导入依赖**:
  - `./useSurveyState.js` - 调查状态 hook
  - `./submitTranscriptShare.js` - 转录分享提交
  - `../../memdir/paths.js` - 内存路径工具
  - `../../tools/FileReadTool/prompt.js` - 文件读取工具常量
  - `../../utils/memoryFileDetection.js` - 内存文件检测

## 备注

- 内存调查专门针对内存功能的使用体验收集反馈
- 20% 的概率比常规反馈调查的 10% 更高，因为内存功能是重要功能
- 转录分享需要用户选择 good/bad 才会触发
