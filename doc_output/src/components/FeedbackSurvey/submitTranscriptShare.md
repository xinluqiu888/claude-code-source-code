# submitTranscriptShare.ts

转录分享提交模块，负责将会话转录上传到服务器。

## 基本信息

- **文件路径**: `src/components/FeedbackSurvey/submitTranscriptShare.ts`
- **导出函数**: submitTranscriptShare
- **类型**: TypeScript 模块

## 功能概述

submitTranscriptShare 负责收集会话数据（包括主会话转录和子代理转录），并将它们安全地上传到 Anthropic 服务器用于产品改进。

## 核心内容

### TranscriptShareResult 类型

```typescript
type TranscriptShareResult = {
  success: boolean
  transcriptId?: string
}
```

### TranscriptShareTrigger 类型

```typescript
type TranscriptShareTrigger = 
  | 'bad_feedback_survey'    // 负面反馈调查
  | 'good_feedback_survey'   // 正面反馈调查
  | 'frustration'            // 用户挫折检测
  | 'memory_survey'          // 内存调查
```

### submitTranscriptShare 函数

```typescript
export async function submitTranscriptShare(
  messages: Message[],
  trigger: TranscriptShareTrigger,
  appearanceId: string,
): Promise<TranscriptShareResult>
```

#### 收集的数据

| 字段 | 说明 |
|------|------|
| trigger | 触发来源 |
| version | Claude Code 版本 |
| platform | 操作系统平台 |
| transcript | 规范化的消息历史 |
| subagentTranscripts | 子代理转录（如果有） |
| rawTranscriptJsonl | 原始 JSONL 转录（大小限制内） |

#### 数据处理流程

1. 收集主会话转录（`normalizeMessagesForAPI`）
2. 收集子代理转录（`loadSubagentTranscripts`）
3. 读取原始 JSONL 转录（大小限制保护）
4. 数据脱敏（`redactSensitiveInfo`）
5. 刷新 OAuth 令牌（如果需要）
6. POST 到 Anthropic API
7. 返回提交结果

#### 大小限制

- 原始转录读取使用 `MAX_TRANSCRIPT_READ_BYTES` 限制，防止 OOM

## 设计要点

- 数据脱敏：使用 `redactSensitiveInfo` 移除敏感信息
- 错误处理：所有错误都被捕获并记录，返回失败状态
- 认证：使用 OAuth 令牌进行 API 认证
- 超时：30 秒 HTTP 请求超时

## 与其他文件的关系

- **导入依赖**:
  - `axios` - HTTP 客户端
  - `fs/promises` - 文件系统操作
  - `../../utils/sessionStorage.js` - 会话存储工具
  - `../Feedback.js` - `redactSensitiveInfo` 脱敏函数

## 备注

- API 端点：`https://api.anthropic.com/api/claude_code_shared_session_transcripts`
- 转录分享完全可选，需要用户明确同意
- 子代理转录帮助理解复杂的多代理会话
