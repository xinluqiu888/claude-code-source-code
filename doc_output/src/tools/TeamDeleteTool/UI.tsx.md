# UI.tsx — 团队删除工具 UI 组件

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TeamDeleteTool/UI.tsx`
- **类型**: TypeScript/React UI 模块
- **功能**: 提供 TeamDelete 工具的 UI 渲染函数

## 核心内容详解

### 导出函数

```typescript
export function renderToolUseMessage(_input: Record<string, unknown>): React.ReactNode

export function renderToolResultMessage(
  content: Output | string,
  _progressMessages: unknown,
  { verbose: _verbose }: { verbose: boolean }
): React.ReactNode
```

### 核心逻辑

#### renderToolUseMessage
返回固定字符串：
```
cleanup team: current
```

#### renderToolResultMessage
始终返回 `null` — 清理结果被抑制显示，因为批量关闭消息已涵盖此信息。

### 实现细节

```typescript
// 如果结果是团队清理输出，返回 null
if ('success' in result && 'team_name' in result && 'message' in result) {
  return null
}
return null
```

## 与其他文件的关系

### 依赖
- `../../utils/slowOperations.js` — JSON 解析工具
- `./TeamDeleteTool.js` — 输出类型定义

### 被依赖
- `TeamDeleteTool.ts` — 工具主实现文件
