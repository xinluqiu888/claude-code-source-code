# UI.tsx — SendMessage工具UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SendMessageTool/UI.tsx`
- **作用**: 渲染SendMessage工具的消息

## 核心内容详解

### 渲染函数

1. **renderToolUseMessage**: 渲染工具使用消息

```typescript
export function renderToolUseMessage(input: Partial<Input>): React.ReactNode
```

处理逻辑：
- 如果message类型为`plan_approval_response`:
  - 批准：`approve plan from: {to}`
  - 拒绝：`reject plan from: {to}`
- 其他情况：返回null

2. **renderToolResultMessage**: 渲染结果消息

```typescript
export function renderToolResultMessage(
  content: SendMessageToolOutput | string,
  _progressMessages: unknown,
  { verbose }: { verbose: boolean }
): React.ReactNode
```

过滤逻辑：
- `routing`字段存在：返回null（内部路由信息）
- `request_id`和`target`字段存在：返回null（请求信息）
- 其他情况：显示`{result.message}`（dimColor）

## 设计要点

1. **条件显示**: 仅在plan_approval_response时显示使用消息
2. **过滤内部信息**: routing和request信息不显示给用户
3. **dimColor**: 结果消息使用dimColor，次要信息弱化
4. **解析支持**: 支持字符串或对象类型的content

## 与其他文件的关系

- **SendMessageTool.ts**: 导入Input和SendMessageToolOutput类型
- **MessageResponse.tsx**: 消息响应组件
- **slowOperations.ts**: `jsonParse`用于解析字符串content
