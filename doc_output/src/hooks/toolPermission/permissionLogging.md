# permissionLogging.ts — 权限决策日志记录

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/toolPermission/permissionLogging.ts`
- **类型**: TypeScript 模块
- **导出函数**: `logPermissionDecision`, `isCodeEditingTool`, `buildCodeEditToolAttributes`
- **导出类型**: `PermissionLogContext`, `PermissionDecisionArgs`
- **依赖**: analytics, OTel telemetry, code-edit metrics

## 功能概述

本模块是权限决策的集中日志记录器，负责：
1. 记录所有权限批准/拒绝事件到分析系统（Statsig）
2. 发送 OTel 遥测事件
3. 跟踪代码编辑工具（Edit/Write/NotebookEdit）的决策计数器
4. 根据决策来源分发到不同的事件名称

## 核心内容详解

### PermissionDecisionArgs 类型

```typescript
type PermissionDecisionArgs =
  | { decision: 'accept'; source: PermissionApprovalSource | 'config' }
  | { decision: 'reject'; source: PermissionRejectionSource | 'config' }
```

### 代码编辑工具定义

```typescript
const CODE_EDITING_TOOLS = ['Edit', 'Write', 'NotebookEdit']
```

### logPermissionDecision 函数

**参数**
- `ctx`: PermissionLogContext - 包含工具、输入、上下文、消息ID、工具使用ID
- `args`: PermissionDecisionArgs - 决策类型和来源
- `permissionPromptStartTimeMs?`: 权限提示开始时间（用于计算等待时间）

**执行流程**
1. 计算用户等待时间（如果有开始时间）
2. 根据决策类型调用 `logApprovalEvent` 或 `logRejectionEvent`
3. 如果是代码编辑工具，构建属性并增加 OTel 计数器
4. 在 `toolUseContext.toolDecisions` Map 中记录决策
5. 发送 OTel 事件

### 批准事件分发

| 来源 | 事件名称 | 额外字段 |
|------|----------|----------|
| config | tengu_tool_use_granted_in_config | 无 |
| classifier | tengu_tool_use_granted_by_classifier | 无 |
| user (permanent) | tengu_tool_use_granted_in_prompt_permanent | 无 |
| user (temporary) | tengu_tool_use_granted_in_prompt_temporary | 无 |
| hook | tengu_tool_use_granted_by_permission_hook | permanent |

### 拒绝事件

统一使用 `tengu_tool_use_rejected_in_prompt`，通过元数据区分：
- hook: `{ isHook: true }`
- user_reject: `{ hasFeedback: boolean }`

### buildCodeEditToolAttributes

为代码编辑工具构建 OTel 计数器属性：
1. 如果工具暴露 `getPath` 方法，提取文件路径
2. 通过 `getLanguageName` 获取语言
3. 返回包含 decision, source, tool_name, language 的属性对象

## 设计要点

### 1. 单一入口

所有权限决策都通过 `logPermissionDecision` 函数记录，确保一致性。

### 2. 决策来源追踪

支持多种决策来源，便于漏斗分析和问题排查。

### 3. 沙盒信息

所有事件都包含 `sandboxEnabled` 字段，用于安全分析。

### 4. 等待时间计算

仅在用户实际被提示时（非自动批准）记录等待时间。

## 与其他文件的关系

- **analytics/index.ts**: 提供 `logEvent`
- **metadata.ts**: 提供 `sanitizeToolNameForAnalytics`
- **state.ts**: 提供 `getCodeEditToolDecisionCounter`
- **cliHighlight.ts**: 提供 `getLanguageName`
- **telemetry/events.ts**: 提供 `logOTelEvent`
- **PermissionContext.ts**: 调用此模块记录决策

## 注意事项

1. **工具名清理**: 使用 `sanitizeToolNameForAnalytics` 确保工具名符合分析要求
2. **异步语言检测**: `buildCodeEditToolAttributes` 是异步的，使用 `.then()` 链式调用
3. **决策持久化**: 决策同时记录在内存 Map 中供下游代码检查
4. **分类器命名**: 当 BASH_CLASSIFIER 或 TRANSCRIPT_CLASSIFIER 启用时，classifier 来源统一标记为 'classifier'
