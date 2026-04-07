# UI.tsx — 网页获取工具 UI 组件

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebFetchTool/UI.tsx`
- **类型**: TypeScript/React UI 模块
- **功能**: 提供 WebFetch 工具的 UI 渲染函数

## 核心内容详解

### 导出函数

```typescript
export function renderToolUseMessage(
  { url, prompt }: Partial<{ url: string; prompt: string }>,
  { verbose }: { verbose: boolean }
): React.ReactNode

export function renderToolUseProgressMessage(): React.ReactNode

export function renderToolResultMessage(
  { bytes, code, codeText, result }: Output,
  _progressMessagesForMessage: ProgressMessage<ToolProgressData>[],
  { verbose }: { verbose: boolean }
): React.ReactNode

export function getToolUseSummary(
  input: Partial<{ url: string; prompt: string }> | undefined
): string | null
```

### 核心逻辑

#### renderToolUseMessage
- 无 URL 时返回 `null`
- Verbose 模式显示 URL 和 prompt
- 非 Verbose 模式仅显示 URL

#### renderToolUseProgressMessage
显示获取进度：
```jsx
<MessageResponse height={1}>
  <Text dimColor>Fetching…</Text>
</MessageResponse>
```

#### renderToolResultMessage
- 显示接收大小和状态码
- Verbose 模式显示完整结果
- 非 Verbose 模式仅显示大小和状态

#### getToolUseSummary
返回截断的 URL 作为工具使用摘要。

## 与其他文件的关系

### 依赖
- `../../components/MessageResponse.js` — 消息响应组件
- `../../utils/format.js` — 格式化工具（文件大小、截断）
- `./WebFetchTool.js` — 输出类型定义

### 被依赖
- `WebFetchTool.ts` — 工具主实现文件
