# envUtils.ts — 环境变量工具函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/envUtils.ts`
- **主要功能**: 环境变量解析、配置目录获取、运行时环境检测
- **关键依赖**: `lodash-es/memoize`, `os`, `path`

## 功能概述

该模块提供环境变量相关的工具函数：
1. Claude 配置目录路径获取
2. 环境变量真值/假值判断
3. 环境变量解析（KEY=VALUE 格式）
4. Node.js 选项检查
5. 裸模式检测
6. AWS/Vertex 区域获取
7. 受保护命名空间检测

## 核心内容详解

### 配置目录

```typescript
export const getClaudeConfigHomeDir = memoize(
  (): string => {
    return (
      process.env.CLAUDE_CONFIG_DIR ?? join(homedir(), '.claude')
    ).normalize('NFC')
  },
  () => process.env.CLAUDE_CONFIG_DIR,
)
```

- 返回 Claude 配置主目录（默认 `~/.claude`）
- 支持 `CLAUDE_CONFIG_DIR` 覆盖
- 使用 NFC Unicode 规范化

### 真值/假值判断

```typescript
export function isEnvTruthy(envVar: string | boolean | undefined): boolean
export function isEnvDefinedFalsy(envVar: string | boolean | undefined): boolean
```

真值: `'1'`, `'true'`, `'yes'`, `'on'`
假值: `'0'`, `'false'`, `'no'`, `'off'`

### 环境变量解析

```typescript
export function parseEnvVars(rawEnvArgs: string[] | undefined): Record<string, string>
// 示例: ['KEY1=value1', 'KEY2=value2'] -> { KEY1: 'value1', KEY2: 'value2' }
```

### 裸模式检测

```typescript
export function isBareMode(): boolean
```

检测 `--bare` 或 `CLAUDE_CODE_SIMPLE` 环境变量。
裸模式跳过：hooks、LSP、插件同步、技能目录遍历、归属、后台预取、凭证读取。

### AWS 区域获取

```typescript
export function getAWSRegion(): string
// 回退顺序: AWS_REGION -> AWS_DEFAULT_REGION -> 'us-east-1'
```

### Vertex 区域获取

```typescript
export function getDefaultVertexRegion(): string
export function getVertexRegionForModel(model): string | undefined
```

支持按模型前缀覆盖区域（如 `claude-opus-4` → `VERTEX_REGION_CLAUDE_4_0_OPUS`）

### 受保护命名空间检测

```typescript
export function isInProtectedNamespace(): boolean
```

检测是否在受保护的 COO 命名空间中（用于遥测）。仅在内部构建中启用。

## 设计要点

1. **memoization**: `getClaudeConfigHomeDir` 缓存 150+ 调用者
2. **大小写不敏感**: 环境变量值判断忽略大小写和空格
3. **保守检测**: `isInProtectedNamespace` 在信号不明确时假设受保护
4. **动态 require**: 内部功能使用条件 require 避免泄漏到外部构建

## 与其他文件的关系

| 文件 | 关系 |
|------|------|
| `protectedNamespace.js` | 内部构建中动态 require |

## 注意事项

1. **配置目录**: 使用 `CLAUDE_CONFIG_DIR` 可在测试中覆盖
2. **区域覆盖**: Vertex 区域按模型前缀优先级匹配
3. **裸模式**: 检查 argv 直接以支持早期门控
4. **Homespace**: `isRunningOnHomespace` 需要 `USER_TYPE=ant` 和 `COO_RUNNING_ON_HOMESPACE`
