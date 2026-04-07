# types.ts — 策略限制类型定义

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/policyLimits/types.ts`
- **作用域**: 策略限制 API 的类型定义和 Zod Schema
- **主要导出**:
  - `PolicyLimitsResponseSchema`: Zod Schema 定义
  - `PolicyLimitsResponse`: API 响应类型
  - `PolicyLimitsFetchResult`: 获取结果类型

## 功能概述

定义策略限制 API 的数据结构和验证 Schema。策略限制 API 返回组织级别的策略限制，仅包含被阻止的策略。如果策略键不存在，则视为允许。

## 核心内容详解

### Zod Schema

```typescript
export const PolicyLimitsResponseSchema = lazySchema(() =>
  z.object({
    restrictions: z.record(z.string(), z.object({ allowed: z.boolean() })),
  }),
)
```

Schema 结构：
- `restrictions`: 记录类型，键为策略名称字符串，值为 `{ allowed: boolean }`

### 类型定义

#### `PolicyLimitsResponse`
API 响应的类型推断：
```typescript
export type PolicyLimitsResponse = z.infer<
  ReturnType<typeof PolicyLimitsResponseSchema>
>
```

#### `PolicyLimitsFetchResult`
获取操作的结果类型：
```typescript
export type PolicyLimitsFetchResult = {
  success: boolean
  restrictions?: PolicyLimitsResponse['restrictions'] | null  // null 表示 304 Not Modified
  etag?: string
  error?: string
  skipRetry?: boolean  // 为 true 时不重试（如认证错误）
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `success` | boolean | 获取是否成功 |
| `restrictions` | Record<string, {allowed: boolean}> \| null \| undefined | 限制列表，null 表示缓存有效（304） |
| `etag` | string \| undefined | HTTP ETag 用于缓存验证 |
| `error` | string \| undefined | 错误消息 |
| `skipRetry` | boolean \| undefined | 是否跳过重试 |

## 设计要点

1. **延迟 Schema**: 使用 `lazySchema` 延迟初始化
2. **宽松设计**: 仅包含被阻止的策略，不存在的策略视为允许
3. **304 处理**: `restrictions` 为 `null` 表示缓存仍然有效

## 与其他文件的关系

- **index.ts**: 使用这些类型和 Schema
- **Zod**: 用于 Schema 定义和验证

## 注意事项

1. **Schema 验证**: API 响应必须通过 Zod Schema 验证
2. **304 语义**: `null` 的 restrictions 有特殊含义（缓存有效）
3. **类型安全**: 使用 Zod 的 `z.infer` 确保类型和 Schema 同步
