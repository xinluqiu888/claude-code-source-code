# commonParameters.ts — 通用参数定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/commonParameters.ts`
- **作用**: PowerShell通用参数定义

## 核心内容详解

### 通用开关参数

```typescript
export const COMMON_SWITCHES = ['-verbose', '-debug']
```

### 通用值参数

```typescript
export const COMMON_VALUE_PARAMS = [
  '-erroraction',
  '-warningaction',
  '-informationaction',
  '-progressaction',
  '-errorvariable',
  '-warningvariable',
  '-informationvariable',
  '-outvariable',
  '-outbuffer',
  '-pipelinevariable',
]
```

### 合并集合

```typescript
export const COMMON_PARAMETERS: ReadonlySet<string> = new Set([
  ...COMMON_SWITCHES,
  ...COMMON_VALUE_PARAMS,
])
```

## 设计要点

1. **标准化格式**: 小写带前导破折号
2. **共享定义**: 被pathValidation.ts和readOnlyValidation.ts使用
3. **打破循环**: 单独文件避免这两个文件间的导入循环

## 与其他文件的关系

- **pathValidation.ts**: 合并到每个cmdlet的已知参数集
- **readOnlyValidation.ts**: 合并到safeFlags检查
