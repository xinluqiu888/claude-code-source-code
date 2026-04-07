# permissions.ts — 权限系统类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/permissions.ts`
- **类型**: TypeScript 模块
- **导出内容**: 权限模式、权限行为、权限规则、权限更新、权限决策等完整类型系统
- **依赖关系**:
  - 导入: `bun:bundle`, `@anthropic-ai/sdk`

## 功能概述

本文件是 Claude Code 权限系统的纯类型定义文件，专门用于打破循环依赖。它包含所有与权限相关的类型定义，从权限模式到权限决策，以及分类器相关类型。这些类型被实现文件导入，而实现文件可以专注于逻辑而不引入循环依赖。

## 核心内容详解

### 1. 权限模式 (Permission Modes)

#### EXTERNAL_PERMISSION_MODES (第16-23行)
```typescript
export const EXTERNAL_PERMISSION_MODES = [
  'acceptEdits',
  'bypassPermissions',
  'default',
  'dontAsk',
  'plan',
] as const
```

**外部可见模式**（5种）:
- `acceptEdits`: 接受编辑模式
- `bypassPermissions`: 绕过权限模式
- `default`: 默认模式
- `dontAsk`: 不询问模式
- `plan`: 计划模式

#### INTERNAL_PERMISSION_MODES (第28-37行)
```typescript
export type InternalPermissionMode = ExternalPermissionMode | 'auto' | 'bubble'
export const INTERNAL_PERMISSION_MODES = [
  ...EXTERNAL_PERMISSION_MODES,
  ...(feature('TRANSCRIPT_CLASSIFIER') ? (['auto'] as const) : ([] as const)),
] as const
```

**内部模式**（7种）:
- 包含所有外部模式
- `auto`: 自动模式（需特性标志开启）
- `bubble`: 气泡模式

**使用说明**:
- `INTERNAL_PERMISSION_MODES`: 用户可寻址的运行时集合
- 用于 settings.json defaultMode、--permission-mode CLI 标志

### 2. 权限行为 (PermissionBehavior)

```typescript
export type PermissionBehavior = 'allow' | 'deny' | 'ask'
```

三种基本权限行为：允许、拒绝、询问

### 3. 权限规则系统

#### PermissionRuleSource (第54-63行)
```typescript
export type PermissionRuleSource =
  | 'userSettings'
  | 'projectSettings'
  | 'localSettings'
  | 'flagSettings'
  | 'policySettings'
  | 'cliArg'
  | 'command'
  | 'session'
```

权限规则来源层级，从用户设置到会话级别。

#### PermissionRuleValue (第67-70行)
```typescript
export type PermissionRuleValue = {
  toolName: string
  ruleContent?: string
}
```

规则值：指定工具名和可选内容匹配。

#### PermissionRule (第75-79行)
```typescript
export type PermissionRule = {
  source: PermissionRuleSource
  ruleBehavior: PermissionBehavior
  ruleValue: PermissionRuleValue
}
```

完整规则定义：来源 + 行为 + 值。

### 4. 权限更新系统

#### PermissionUpdateDestination (第88-94行)
```typescript
export type PermissionUpdateDestination =
  | 'userSettings'
  | 'projectSettings'
  | 'localSettings'
  | 'session'
  | 'cliArg'
```

更新操作的目标持久化位置。

#### PermissionUpdate (第98-131行)

联合类型，支持多种更新操作：

| 操作类型 | 说明 |
|----------|------|
| `addRules` | 添加规则到指定目标 |
| `replaceRules` | 替换规则 |
| `removeRules` | 移除规则 |
| `setMode` | 设置模式 |
| `addDirectories` | 添加额外工作目录 |
| `removeDirectories` | 移除额外工作目录 |

### 5. 工作目录权限

#### AdditionalWorkingDirectory (第143-146行)
```typescript
export type AdditionalWorkingDirectory = {
  path: string
  source: WorkingDirectorySource
}
```

额外包含在权限范围中的目录及其来源。

### 6. 权限决策系统

#### PermissionCommandMetadata (第157-162行)
```typescript
export type PermissionCommandMetadata = {
  name: string
  description?: string
  [key: string]: unknown
}
```

最小化的命令元数据，用于权限决策，避免完整 Command 类型的循环依赖。

#### PermissionAllowDecision (第174-184行)
```typescript
export type PermissionAllowDecision<Input extends { [key: string]: unknown } = { [key: string]: unknown }> = {
  behavior: 'allow'
  updatedInput?: Input
  userModified?: boolean
  decisionReason?: PermissionDecisionReason
  toolUseID?: string
  acceptFeedback?: string
  contentBlocks?: ContentBlockParam[]
}
```

**允许决策**包含：
- 可选更新输入
- 用户修改标志
- 决策原因
- 工具使用ID
- 接受反馈
- 内容块（支持图片等）

#### PermissionAskDecision (第199-226行)

**询问决策**包含：
- 显示消息
- 建议的更新操作
- 阻塞路径
- 元数据
- 待处理分类器检查（异步自动批准）
- 内容块
- 特殊标志：`isBashSecurityCheckForMisparsing`

#### PermissionDenyDecision (第231-236行)

**拒绝决策**包含：
- 拒绝消息
- 决策原因（必需）
- 工具使用ID

#### PermissionDecision (第241-246行)

三种决策的联合类型。

#### PermissionResult (第251-266行)

扩展决策结果，增加 `passthrough` 选项，用于权限代理场景。

### 7. PermissionDecisionReason (第271-324行)

决策原因的详细分型：

| 类型 | 用途 |
|------|------|
| `rule` | 规则匹配 |
| `mode` | 模式决定 |
| `subcommandResults` | 子命令结果 |
| `permissionPromptTool` | 权限提示工具 |
| `hook` | Hook 决策 |
| `asyncAgent` | 异步代理 |
| `sandboxOverride` | 沙箱覆盖 |
| `classifier` | 分类器决策 |
| `workingDir` | 工作目录检查 |
| `safetyCheck` | 安全检查（含 `classifierApprovable` 标志） |
| `other` | 其他原因 |

### 8. Bash 分类器类型 (第330-397行)

#### ClassifierResult (第330-335行)
```typescript
export type ClassifierResult = {
  matches: boolean
  matchedDescription?: string
  confidence: 'high' | 'medium' | 'low'
  reason: string
}
```

#### ClassifierBehavior (第337行)
```typescript
export type ClassifierBehavior = 'deny' | 'ask' | 'allow'
```

#### ClassifierUsage (第339-344行)

分类器使用的 token 统计。

#### YoloClassifierResult (第346-397行)

丰富的分类器结果，包含：
- 思考内容
- 是否应阻止
- 原因
- 不可用标志
- 模型信息
- Token 使用统计（单阶段/双阶段）
- 请求ID和消息ID（用于服务端关联）
- 提示长度统计
- 错误转储路径

### 9. PermissionExplainer 类型 (第403-410行)

```typescript
export type PermissionExplanation = {
  riskLevel: 'LOW' | 'MEDIUM' | 'HIGH'
  explanation: string
  reasoning: string
  risk: string
}
```

权限解释的风险等级和说明。

### 10. ToolPermissionContext (第416-441行)

工具权限检查的完整上下文：

```typescript
export type ToolPermissionContext = {
  readonly mode: PermissionMode
  readonly additionalWorkingDirectories: ReadonlyMap<string, AdditionalWorkingDirectory>
  readonly alwaysAllowRules: ToolPermissionRulesBySource
  readonly alwaysDenyRules: ToolPermissionRulesBySource
  readonly alwaysAskRules: ToolPermissionRulesBySource
  readonly isBypassPermissionsModeAvailable: boolean
  readonly strippedDangerousRules?: ToolPermissionRulesBySource
  readonly shouldAvoidPermissionPrompts?: boolean
  readonly awaitAutomatedChecksBeforeDialog?: boolean
  readonly prePlanMode?: PermissionMode
}
```

**关键字段**:
- `mode`: 当前权限模式
- `additionalWorkingDirectories`: 额外工作目录映射
- 三类规则映射（allow/deny/ask）
- 各种行为标志

## 设计要点

1. **纯类型文件**: 无运行时依赖，专门用于打破循环导入
2. **泛型决策**: 决策类型支持泛型输入类型
3. **丰富元数据**: 决策原因包含详细的上下文信息
4. **分类器统计**: YoloClassifierResult 包含详细的指标用于分析
5. **只读上下文**: ToolPermissionContext 使用 readonly 修饰符

## 与其他文件的关系

- **utils/permissions/**: 实现文件从这里导入类型
- **Tool.ts**: 使用 ToolPermissionContext
- **hooks.ts**: 使用 PermissionResult 等类型

## 注意事项

1. **特性标志**: `auto` 模式需要 `TRANSCRIPT_CLASSIFIER` 特性标志
2. **泛型默认**: PermissionDecision 默认接受任意对象输入
3. **前向兼容**: PermissionCommandMetadata 允许额外属性
4. **分类器两阶段**: 支持 fast/thinking 两阶段分类
