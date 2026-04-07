# UI.tsx — PowerShell工具UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/UI.tsx`
- **作用**: 渲染PowerShell工具的消息

## 核心内容详解

### 常量

```typescript
const MAX_COMMAND_DISPLAY_LINES = 2
const MAX_COMMAND_DISPLAY_CHARS = 160
```

### 渲染函数

1. **renderToolUseMessage**: 渲染命令

```typescript
export function renderToolUseMessage(
  input: Partial<PowerShellToolInput>,
  { verbose, theme }: { verbose: boolean; theme: ThemeName }
): React.ReactNode
```

- 非verbose: 截断到2行或160字符
- verbose: 显示完整命令

2. **renderToolUseProgressMessage**: 渲染进度

```typescript
export function renderToolUseProgressMessage(
  progressMessagesForMessage: ProgressMessage<PowerShellProgress>[],
  { verbose, tools, terminalSize, inProgressToolCallCount }
): React.ReactNode
```

使用`ShellProgressMessage`组件显示执行进度。

3. **renderToolUseQueuedMessage**: 渲染队列状态

```typescript
export function renderToolUseQueuedMessage(): React.ReactNode
```

显示"Waiting..."等待状态。

4. **renderToolResultMessage**: 渲染结果

```typescript
export function renderToolResultMessage(
  content: Out,
  progressMessagesForMessage: ProgressMessage<PowerShellProgress>[],
  { verbose, theme, tools, style }
): React.ReactNode
```

显示：
- stdout输出
- stderr错误（红色）
- 空输出提示
- 超时显示
- 背景任务指示

5. **renderToolUseErrorMessage**: 渲染错误

使用`FallbackToolUseErrorMessage`组件。

## 设计要点

1. **智能截断**: 长命令自动截断
2. **进度显示**: 实时显示执行进度
3. **空输出处理**: 区分无输出、中断、后台任务
4. **超时指示**: 显示ShellTimeDisplay

## 与其他文件的关系

- **PowerShellTool.tsx**: 导入渲染函数
- **components/shell/ShellProgressMessage.tsx**: 进度组件
- **components/shell/ShellTimeDisplay.tsx**: 时间显示组件
