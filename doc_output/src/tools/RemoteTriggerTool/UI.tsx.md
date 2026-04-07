# UI.tsx — RemoteTrigger工具UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/RemoteTriggerTool/UI.tsx`
- **作用**: 渲染RemoteTrigger工具的消息

## 核心内容详解

### 渲染函数

1. **renderToolUseMessage**: 渲染工具使用消息

```typescript
export function renderToolUseMessage(input: Partial<Input>): React.ReactNode {
  return `${input.action ?? ''}${input.trigger_id ? ` ${input.trigger_id}` : ''}`
}
```

返回格式：`{action} {trigger_id}`（如："list"、"get my-trigger"）

2. **renderToolResultMessage**: 渲染结果消息

```typescript
export function renderToolResultMessage(output: Output): React.ReactNode
```

显示：
- HTTP状态码
- 响应JSON行数（如："HTTP 200 (15 lines)"）

## 设计要点

1. **简洁显示**: 工具使用消息仅显示action和trigger_id
2. **行数统计**: 结果消息显示JSON响应的行数，便于快速了解响应大小
3. **dimColor**: 使用dimColor显示行数，次要信息弱化

## 与其他文件的关系

- **RemoteTriggerTool.ts**: 导入Input和Output类型
- **MessageResponse.tsx**: 消息响应组件
- **stringUtils.ts**: `countCharInString`用于统计行数
