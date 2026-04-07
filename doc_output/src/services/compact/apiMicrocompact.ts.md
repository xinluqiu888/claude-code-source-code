# apiMicrocompact.ts — API 原生微压缩配置

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/apiMicrocompact.ts`
- **作用域**: API 层上下文管理策略配置
- **主要导出**:
  - `getAPIContextManagement`: 获取 API 上下文管理配置
  - `ContextManagementConfig`: 配置类型定义
  - `ContextEditStrategy`: 编辑策略类型

## 功能概述

提供基于 API 的原生上下文管理功能，支持工具使用清除和 thinking 块清除策略。这是服务端微压缩的实现，通过 API 参数控制上下文编辑行为。

## 核心内容详解

### 策略类型

```typescript
export type ContextEditStrategy =
  | {
      type: 'clear_tool_uses_20250919'
      trigger?: { type: 'input_tokens'; value: number }
      keep?: { type: 'tool_uses'; value: number }
      clear_tool_inputs?: boolean | string[]
      exclude_tools?: string[]
    }
  | {
      type: 'clear_thinking_20251015'
      keep: { type: 'thinking_turns'; value: number } | 'all'
    }
```

### 主要功能

#### `getAPIContextManagement`
根据当前环境和配置返回上下文管理策略：

1. **Thinking 清除策略**:
   - 当存在 thinking 块且未启用 redact-thinking 时
   - 支持保留所有 thinking 或仅保留最后一轮

2. **工具结果清除策略** (`ant` 用户专用):
   - `USE_API_CLEAR_TOOL_RESULTS`: 清除指定工具的结果
   - `USE_API_CLEAR_TOOL_USES`: 保留指定工具，清除其他

### 默认配置

```typescript
const DEFAULT_MAX_INPUT_TOKENS = 180_000  // 警告阈值
const DEFAULT_TARGET_INPUT_TOKENS = 40_000 // 保留最近 40k tokens
```

### 可清除工具列表

```typescript
const TOOLS_CLEARABLE_RESULTS = [
  ...SHELL_TOOL_NAMES,
  GLOB_TOOL_NAME,
  GREP_TOOL_NAME,
  FILE_READ_TOOL_NAME,
  WEB_FETCH_TOOL_NAME,
  WEB_SEARCH_TOOL_NAME,
]

const TOOLS_CLEARABLE_USES = [
  FILE_EDIT_TOOL_NAME,
  FILE_WRITE_TOOL_NAME,
  NOTEBOOK_EDIT_TOOL_NAME,
]
```

## 设计要点

1. **环境感知**: 根据 `USER_TYPE` 和 feature flags 启用不同策略
2. **阈值配置**: 支持通过环境变量覆盖默认阈值
3. **Ant 专用**: 工具清除策略仅对 `ant` 用户启用

## 与其他文件的关系

- **claude.ts**: 调用 `getAPIContextManagement` 构建 API 请求参数
- **工具常量文件**: 引用各工具的常量定义

## 注意事项

1. **API 版本**: 策略类型包含版本日期（如 `20250919`）
2. **环境变量**: 
   - `USE_API_CLEAR_TOOL_RESULTS`
   - `USE_API_CLEAR_TOOL_USES`
   - `API_MAX_INPUT_TOKENS`
   - `API_TARGET_INPUT_TOKENS`
