# textInputTypes.ts — 文本输入组件类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/textInputTypes.ts`
- **类型**: TypeScript 模块
- **导出内容**: 文本输入组件 Props、状态、输入模式、队列命令等完整类型
- **依赖关系**:
  - 导入: `@anthropic-ai/sdk`, `crypto`, `React`, `agentSdkTypes.js`, `ink.js`, `config.js`, `imageResizer.js`, `textHighlighting.js`, `ids.js`, `message.js`

## 功能概述

本文件定义了 Claude Code 文本输入系统的完整类型体系，包括基础文本输入、Vim 模式输入、提示输入的各种模式和状态。它支持复杂的多行输入、粘贴处理、历史导航、幽灵文本等功能，是 REPL 输入组件的类型基础。

## 核心内容详解

### 1. InlineGhostText (第15-23行)

内联幽灵文本类型，用于命令自动完成的中间提示：

```typescript
export type InlineGhostText = {
  readonly text: string           // 显示文本（如 /commit 的 "mit"）
  readonly fullCommand: string    // 完整命令名（如 "commit"）
  readonly insertPosition: number // 插入位置
}
```

### 2. BaseTextInputProps (第27-203行)

文本输入组件的基础属性（约 50 个属性）：

**核心属性**:
- `value`: 当前值
- `onChange`: 值变化回调
- `onSubmit`: 提交回调
- `placeholder`: 占位符文本

**行为控制**:
- `multiline`: 允许多行输入（默认 true）
- `mask`: 掩码字符（用于密码输入）
- `showCursor`: 显示光标
- `highlightPastedText`: 高亮粘贴文本
- `focus`: 是否获得焦点
- `dimColor`: 是否使用暗淡颜色

**事件处理**:
- `onHistoryUp/Down`: 历史导航
- `onExit`: Ctrl+C 退出
- `onExitMessage`: 显示退出消息
- `onClearInput`: 清空输入
- `onUndo`: 撤销功能

**图片粘贴**:
```typescript
onImagePaste?: (
  base64Image: string,
  mediaType?: string,
  filename?: string,
  dimensions?: ImageDimensions,
  sourcePath?: string,
) => void
```

**文本粘贴**:
- `onPaste`: 大文本（>800字符）粘贴处理
- `onIsPastingChange`: 粘贴状态变化

**显示控制**:
- `columns`: 列数（自动换行）
- `maxVisibleLines`: 最大可见行数
- `highlights`: 文本高亮（搜索结果等）
- `placeholderElement`: 自定义占位符 React 元素
- `inlineGhostText`: 内联幽灵文本
- `inputFilter`: 输入过滤器

**特殊控制**:
- `disableCursorMovementForUpDownKeys`: 禁用上下键光标移动
- `disableEscapeDoublePress`: 禁用双击 Escape 处理
- `cursorOffset`: 光标偏移
- `onChangeCursorOffset`: 光标偏移变化
- `argumentHint`: 参数提示文本

### 3. VimTextInputProps (第207-217行)

Vim 模式输入的扩展属性：

```typescript
export type VimTextInputProps = BaseTextInputProps & {
  readonly initialMode?: VimMode      // 初始模式
  readonly onModeChange?: (mode: VimMode) => void  // 模式变化回调
}
```

### 4. VimMode (第222行)

```typescript
export type VimMode = 'INSERT' | 'NORMAL'
```

Vim 编辑器两种基本模式。

### 5. BaseInputState (第227-247行)

输入状态的基础定义：

```typescript
export type BaseInputState = {
  onInput: (input: string, key: Key) => void
  renderedValue: string
  offset: number
  setOffset: (offset: number) => void
  cursorLine: number          // 光标所在行（0索引）
  cursorColumn: number        // 光标所在列（显示宽度）
  viewportCharOffset: number  // 视口起始偏移
  viewportCharEnd: number     // 视口结束偏移
  isPasting?: boolean
  pasteState?: {
    chunks: string[]
    timeoutId: ReturnType<typeof setTimeout> | null
  }
}
```

### 6. TextInputState & VimInputState (第252-260行)

```typescript
export type TextInputState = BaseInputState
export type VimInputState = BaseInputState & {
  mode: VimMode
  setMode: (mode: VimMode) => void
}
```

### 7. PromptInputMode (第265-270行)

提示输入的输入模式：

```typescript
export type PromptInputMode =
  | 'bash'                   // Bash 命令
  | 'prompt'                 // 普通提示
  | 'orphaned-permission'    // 孤儿权限
  | 'task-notification'      // 任务通知
```

### 8. QueuePriority (第278-294行)

队列优先级级别：

```typescript
export type QueuePriority = 'now' | 'next' | 'later'
```

**语义**:
- `now`: 立即中断发送，中止当前工具调用
- `next`: 中回合耗尽，当前工具完成后发送
- `later`: 回合结束耗尽，等待当前回合完成

### 9. QueuedCommand (第299-358行)

队列命令的完整定义：

**基础字段**:
- `value`: 值（字符串或 ContentBlockParam 数组）
- `mode`: 输入模式
- `priority`: 优先级（可选，默认根据模式推断）
- `uuid`: UUID
- `orphanedPermission`: 孤儿权限

**粘贴相关**:
- `pastedContents`: 原始粘贴内容（包含图片）
- `preExpansionValue`: 粘贴占位符展开前的原始输入

**命令控制**:
- `skipSlashCommands`: 是否跳过斜杠命令
- `bridgeOrigin`: 是否来自桥接（过滤危险命令）
- `isMeta`: 是否标记为元消息
- `origin`: 来源（记录到消息中）
- `workload`: 工作负载标签
- `agentId`: 目标代理（用于子代理隔离）

### 10. isValidImagePaste() (第367-369行)

检查粘贴内容是否为有效的图片：

```typescript
export function isValidImagePaste(c: PastedContent): boolean {
  return c.type === 'image' && c.content.length > 0
}
```

**用途**: 过滤空内容的图片（如0字节文件拖拽），避免 API 返回 "image cannot be empty" 错误。

### 11. getImagePasteIds() (第372-382行)

从 QueuedCommand 的 pastedContents 中提取有效的图片 ID：

```typescript
export function getImagePasteIds(
  pastedContents: Record<number, PastedContent> | undefined,
): number[] | undefined
```

**逻辑**:
1. 检查 pastedContents 是否存在
2. 过滤出有效的图片（使用 isValidImagePaste）
3. 映射到 ID 数组
4. 空数组返回 undefined

### 12. OrphanedPermission (第384-388行)

孤儿权限类型：

```typescript
export type OrphanedPermission = {
  permissionResult: PermissionResult
  assistantMessage: AssistantMessage
}
```

表示与工具使用分离的权限结果，通常在恢复或特殊场景下出现。

## 设计要点

1. **属性丰富**: BaseTextInputProps 包含约 50 个属性，支持丰富的定制
2. **类型安全**: 使用 readonly 修饰符确保不可变性
3. **队列语义**: QueuePriority 明确定义了三种处理时机
4. **粘贴处理**: 专门处理图片和文本粘贴，包括验证和 ID 提取
5. **Vim 支持**: 完整的 Vim 模式输入支持

## 与其他文件的关系

- **components/**: 文本输入组件使用这些类型
- **hooks/useTextInput.ts**: 实现输入状态管理
- **types/message.ts**: QueuedCommand 与 Message 关联

## 使用场景

```typescript
// 基础文本输入
const props: BaseTextInputProps = {
  value: inputValue,
  onChange: setInputValue,
  onSubmit: handleSubmit,
  columns: 80,
  cursorOffset: cursor,
  onChangeCursorOffset: setCursor,
}

// Vim 输入
const vimProps: VimTextInputProps = {
  ...baseProps,
  initialMode: 'NORMAL',
  onModeChange: handleModeChange,
}

// 队列命令
const command: QueuedCommand = {
  value: '/commit',
  mode: 'prompt',
  priority: 'now',
}
```

## 注意事项

1. **光标位置**: cursorOffset 是字符偏移，cursorColumn 是显示宽度
2. **粘贴状态**: 大文本粘贴有专门的 timeout 状态管理
3. **桥接过滤**: bridgeOrigin 标志用于过滤桥接来源的危险命令
4. **图片验证**: 必须检查图片内容非空，否则 API 会报错
