# slowOperations.ts — 慢操作检测和包装

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/slowOperations.ts`
- **主要功能**: 检测和记录慢操作，包装常用函数添加性能监控
- **关键依赖**: `bun:bundle`, `lodash-es/cloneDeep`, `bootstrap/state.js`, `debug.ts`

## 功能概述

该模块提供慢操作检测机制：
1. 检测超过阈值的 JSON 序列化/克隆操作
2. 使用 `using` 声明的模板字符串标记
3. 包装常用函数（JSON.stringify, JSON.parse, structuredClone, cloneDeep）
4. 构建特定的慢日志（仅在内部构建中启用）

## 核心内容详解

### 阈值配置

```typescript
const SLOW_OPERATION_THRESHOLD_MS = (() => {
  const envValue = process.env.CLAUDE_CODE_SLOW_OPERATION_THRESHOLD_MS
  if (envValue !== undefined) {
    const parsed = Number(envValue)
    if (!Number.isNaN(parsed) && parsed >= 0) {
      return parsed
    }
  }
  if (process.env.NODE_ENV === 'development') {
    return 20  // 开发模式：20ms
  }
  if (process.env.USER_TYPE === 'ant') {
    return 300  // 内部用户：300ms
  }
  return Infinity  // 外部用户：禁用
})()
```

### 慢日志模板

```typescript
export const slowLogging: {
  (strings: TemplateStringsArray, ...values: unknown[]): Disposable
}
```

使用 tagged template 标记操作：

```typescript
using _ = slowLogging`JSON.stringify(${value})`
const result = JSON.stringify(value)
```

### AntSlowLogger 类

```typescript
class AntSlowLogger {
  startTime: number
  args: IArguments
  err: Error

  [Symbol.dispose](): void {
    const duration = performance.now() - this.startTime
    if (duration > SLOW_OPERATION_THRESHOLD_MS && !isLogging) {
      // 记录慢操作到调试日志和状态
    }
  }
}
```

- 使用 `Symbol.dispose` 配合 `using` 声明
- 构造函数时记录开始时间
- dispose 时检查持续时间

### 包装函数

#### jsonStringify

```typescript
export function jsonStringify(
  value: unknown,
  replacer?: ...,
  space?: string | number
): string
```

包装 `JSON.stringify`，添加慢操作检测。

#### jsonParse

```typescript
export const jsonParse: typeof JSON.parse
```

包装 `JSON.parse`，添加慢操作检测。
- 显式分支处理无 reviver 的情况（V8 优化）

#### clone

```typescript
export function clone<T>(value: T, options?: StructuredSerializeOptions): T
```

包装 `structuredClone`。

#### cloneDeep

```typescript
export function cloneDeep<T>(value: T): T
```

包装 lodash 的 `cloneDeep`。

#### writeFileSync_DEPRECATED

```typescript
export function writeFileSync_DEPRECATED(
  filePath: string,
  data: string | NodeJS.ArrayBufferView,
  options?: WriteFileOptionsWithFlush
): void
```

支持 `flush` 选项的同步文件写入。
- 标记为 DEPRECATED，建议使用异步 API
- 需要 flush 时手动实现（open, write, fsync, close）

### 辅助函数

#### callerFrame

```typescript
export function callerFrame(stack: string | undefined): string
```

提取调用者位置信息，用于 DevBar 警告。

#### buildDescription

```typescript
function buildDescription(args: IArguments): string
```

从模板参数构建可读描述：
- 数组显示长度
- 对象显示键数量
- 字符串截断到80字符

## 设计要点

1. **条件编译**: 使用 `feature('SLOW_OPERATION_LOGGING')` 控制是否启用
2. **零开销**: 外部构建返回空 disposable，无性能影响
3. **Using 声明**: 利用 TypeScript 的 `using` 自动触发检测
4. **死代码消除**: AntSlowLogger 在外部构建中被 DCE 移除
5. **递归保护**: 模块级 `isLogging` 防止日志记录时的递归

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `bootstrap/state.js` | 使用 `addSlowOperation` |
| `debug.ts` | 使用 `logForDebugging` |
| `lodash-es/cloneDeep` | 导入用于 `cloneDeep` |

## 注意事项

1. **阈值环境变量**: `CLAUDE_CODE_SLOW_OPERATION_THRESHOLD_MS` 可覆盖默认阈值
2. **V8 优化**: `jsonParse` 显式分支保持无 reviver 路径优化
3. **已废弃**: `writeFileSync_DEPRECATED` 不应在新代码中使用
4. **性能**: 仅在内部构建中启用，外部构建零开销
5. **调用栈**: 通过构造时的 Error 捕获调用栈
