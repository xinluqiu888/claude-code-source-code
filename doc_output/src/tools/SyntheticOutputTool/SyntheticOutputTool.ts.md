# SyntheticOutputTool.ts — 结构化输出工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SyntheticOutputTool/SyntheticOutputTool.ts`
- **作用**: 以结构化JSON格式返回最终结果

## 功能概述

该工具仅在非交互式会话中启用（SDK/CLI使用），允许模型以指定的JSON schema格式返回结构化输出。工具会验证输出是否符合schema要求。

## 核心内容详解

### 工具名称常量

```typescript
export const SYNTHETIC_OUTPUT_TOOL_NAME = 'StructuredOutput'
```

### 启用条件

```typescript
export function isSyntheticOutputToolEnabled(opts: {
  isNonInteractiveSession: boolean
}): boolean {
  return opts.isNonInteractiveSession
}
```

仅在非交互式会话中启用（SDK/CLI模式）。

### Schema定义

**输入Schema**:
```typescript
z.object({}).passthrough() // 允许任何输入对象
```

动态schema通过`createSyntheticOutputTool`提供。

**输出Schema**:
```typescript
z.string().describe('Structured output tool result')
```

### 基础工具定义

```typescript
export const SyntheticOutputTool = buildTool({
  isMcp: false,
  isEnabled: () => true, // 创建后始终启用
  isConcurrencySafe: () => true,
  isReadOnly: () => true,
  isOpenWorld: () => false,
  name: SYNTHETIC_OUTPUT_TOOL_NAME,
  // ...
})
```

### 创建带Schema的工具

```typescript
export function createSyntheticOutputTool(
  jsonSchema: Record<string, unknown>
): CreateResult
```

工作流程：
1. 检查缓存（使用WeakMap避免重复编译）
2. 使用Ajv验证schema有效性
3. 编译schema为验证函数
4. 返回配置好的工具

**缓存机制**:
```typescript
const toolCache = new WeakMap<object, CreateResult>()
```

工作流程脚本每次调用`agent({schema: BUGS_SCHEMA})`时，80次调用从~110ms Ajv开销降到~4ms。

### 验证逻辑

```typescript
async call(input) {
  const isValid = validateSchema(input)
  if (!isValid) {
    const errors = validateSchema.errors
      ?.map(e => `${e.instancePath || 'root'}: ${e.message}`)
      .join(', ')
    throw new TelemetrySafeError(...)
  }
  return {
    data: 'Structured output provided successfully',
    structured_output: input
  }
}
```

## 设计要点

1. **非交互式专用**: 仅在SDK/CLI会话中启用
2. **动态Schema**: Schema在运行时通过`createSyntheticOutputTool`提供
3. **Ajv验证**: 使用Ajv进行JSON Schema验证
4. **WeakMap缓存**: 相同schema复用编译结果，优化性能
5. **只读操作**: `isReadOnly: true`
6. **并发安全**: `isConcurrencySafe: true`

## 与其他文件的关系

- **Tool.ts**: Tool接口和buildTool
- **ajv**: JSON Schema验证库
- **errors.ts**: TelemetrySafeError

## 注意事项

- 该工具在main.tsx中根据`isSyntheticOutputToolEnabled()`条件创建
- 一旦创建始终启用（条件在创建时检查）
- 验证错误使用TelemetrySafeError，确保敏感信息不泄露到遥测
- 返回的`structured_output`是实际的结构化数据
