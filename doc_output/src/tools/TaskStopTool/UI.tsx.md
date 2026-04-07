# UI.tsx — 任务停止工具 UI 组件

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TaskStopTool/UI.tsx`
- **类型**: TypeScript/React UI 模块
- **功能**: 提供 TaskStop 工具的 UI 渲染函数

## 核心内容详解

### 导出函数

```typescript
export function renderToolUseMessage(): React.ReactNode

export function renderToolResultMessage(
  output: Output,
  _progressMessagesForMessage: unknown[],
  { verbose }: { verbose: boolean }
): React.ReactNode
```

### 核心逻辑

#### renderToolUseMessage
返回空字符串 `''`。

#### renderToolResultMessage
显示停止的任务命令：
```
{command} · stopped
```

### 命令截断
- 最大显示行数: 2 行
- 最大显示字符: 160 字符
- 使用 `truncateToWidthNoEllipsis` 进行截断

### 用户类型检查
```typescript
if ("external" === 'ant') {
  return null
}
```

## 与其他文件的关系

### 依赖
- `../../utils/format.js` — 格式化工具
- `../../utils/slowOperations.js` — JSON 解析工具
- `./TaskStopTool.js` — 输出类型定义

### 被依赖
- `TaskStopTool.ts` — 工具主实现文件
