# coreTypes.ts — SDK 核心类型

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/sdk/coreTypes.ts`
- **类型**: TypeScript 类型定义
- **语言**: TypeScript

## 功能概述

SDK 核心类型定义文件，包含可被 SDK 消费者和构建者共同使用的可序列化类型。从 coreSchemas.ts 的 Zod schema 生成。

## 核心内容详解

### 沙箱类型重导出

```typescript
export type {
  SandboxFilesystemConfig,
  SandboxIgnoreViolations,
  SandboxNetworkConfig,
  SandboxSettings,
} from '../sandboxTypes.js'
```

### 生成的类型重导出

```typescript
export * from './coreTypes.generated.js'
```

所有核心类型从生成的文件导出，基于 coreSchemas.ts 中的 Zod schema 自动生成。

### 工具类型重导出

```typescript
export type { NonNullableUsage } from './sdkUtilityTypes.js'
```

### 常量数组

1. **HOOK_EVENTS** — Hook 事件类型
   - PreToolUse, PostToolUse, PostToolUseFailure
   - Notification, UserPromptSubmit
   - SessionStart, SessionEnd
   - Stop, StopFailure
   - SubagentStart, SubagentStop
   - PreCompact, PostCompact
   - PermissionRequest, PermissionDenied
   - Setup, TeammateIdle
   - TaskCreated, TaskCompleted
   - Elicitation, ElicitationResult
   - ConfigChange
   - WorktreeCreate, WorktreeRemove
   - InstructionsLoaded
   - CwdChanged, FileChanged

2. **EXIT_REASONS** — 退出原因
   - clear, resume, logout
   - prompt_input_exit, other
   - bypass_permissions_disabled

## 设计要点

1. **类型生成工作流**:
   - 类型从 Zod schema 自动生成
   - 修改流程:
     1. 编辑 coreSchemas.ts 中的 Zod schema
     2. 运行: `bun scripts/generate-sdk-types.ts`

2. **运行时 vs 类型**:
   - Schema 在 coreSchemas.ts 中用于运行时验证
   - Schema 不是公共 API 的一部分
   - 类型在此文件中导出供公共使用

3. **联合类型**:
   - HOOK_EVENTS 和 EXIT_REASONS 使用 const 数组
   - 支持运行时遍历和类型安全

4. **沙箱类型**:
   - 从 sandboxTypes.ts 重新导出
   - 确保单一真相源

## 与其他文件的关系

- **./coreSchemas.ts**: Zod schema 定义
- **./coreTypes.generated.ts**: 生成的类型文件
- **../sandboxTypes.ts**: 沙箱配置类型
- **./sdkUtilityTypes.ts**: 工具类型

## 注意事项

1. 不要直接编辑 coreTypes.ts 中的类型
2. 所有类型修改应通过编辑 coreSchemas.ts 完成
3. 生成脚本会覆盖 coreTypes.generated.ts
4. Schema 可用于运行时验证但不属于公共 API
5. HOOK_EVENTS 和 EXIT_REASONS 在运行时可用
