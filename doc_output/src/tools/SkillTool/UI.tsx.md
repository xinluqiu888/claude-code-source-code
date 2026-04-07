# UI.tsx — Skill工具UI

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SkillTool/UI.tsx`
- **作用**: 渲染Skill工具的消息和进度

## 核心内容详解

### 常量

```typescript
const MAX_PROGRESS_MESSAGES_TO_SHOW = 3
const INITIALIZING_TEXT = 'Initializing…'
```

### 渲染函数

1. **renderToolResultMessage**: 渲染结果消息

```typescript
export function renderToolResultMessage(output: Output): React.ReactNode
```

处理逻辑：
- forked状态：显示"Done"
- 普通状态：显示"Successfully loaded skill"
  - 如果有allowedTools：显示工具数量
  - 如果有model：显示模型名称

2. **renderToolUseMessage**: 渲染工具使用消息

```typescript
export function renderToolUseMessage(
  { skill }: Partial<Input>,
  { commands }: { commands?: Command[] }
): React.ReactNode
```

处理逻辑：
- 查找命令，检查是否来自`commands_DEPRECATED`目录
- 如果是：显示`/{skill}`
- 否则：显示`{skill}`

3. **renderToolUseProgressMessage**: 渲染进度消息

```typescript
export function renderToolUseProgressMessage(
  progressMessages: ProgressMessage<Progress>[],
  { tools, verbose }: { tools: Tools; verbose: boolean }
): React.ReactNode
```

显示逻辑：
- 无进度：显示"Initializing…"
- 有进度：显示最近3条（非verbose模式）
- 使用SubAgentProvider包裹消息组件
- 超出3条显示"+{count} more tool uses"

4. **renderToolUseRejectedMessage**: 渲染拒绝消息

```typescript
export function renderToolUseRejectedMessage(...): React.ReactNode
```

显示进度消息 + FallbackToolUseRejectedMessage

5. **renderToolUseErrorMessage**: 渲染错误消息

```typescript
export function renderToolUseErrorMessage(...): React.ReactNode
```

显示进度消息 + FallbackToolUseErrorMessage

## 设计要点

1. **forked状态处理**: 区分inline和forked执行状态
2. **进度显示**: 最多显示3条进度消息，避免刷屏
3. **SubAgentProvider**: 子代理消息使用专用Provider
4. **历史兼容**: 识别commands_DEPRECATED目录的技能
5. **渐进信息**: 根据verbose模式调整显示详细程度

## 与其他文件的关系

- **SkillTool.ts**: 导入Input, Output, Progress类型
- **Message.tsx**: 消息组件
- **CtrlOToExpand.tsx**: SubAgentProvider
- **FallbackToolUseRejectedMessage.tsx**: 拒绝回退组件
- **FallbackToolUseErrorMessage.tsx**: 错误回退组件
- **messages.ts**: `buildSubagentLookups`

## 注意事项

- 进度消息使用condensed样式，减少视觉噪音
- forked执行显示简化结果（仅"Done"）
- 技能名称显示格式反映来源（传统/现代）
