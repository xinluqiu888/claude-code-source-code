# bundledMode.ts — 运行时环境检测

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/bundledMode.ts`
- **主要功能**: 检测 Bun 运行时和独立可执行文件模式
- **关键依赖**: 无

## 功能概述

该模块提供两个简单的运行时检测函数：
1. 检测是否在 Bun 环境下运行
2. 检测是否作为 Bun 编译的独立可执行文件运行

## 核心内容详解

### isRunningWithBun

```typescript
export function isRunningWithBun(): boolean
```

检测是否在 Bun 环境下运行。

返回 `true` 当：
- 通过 `bun` 命令运行 JS 文件
- 运行 Bun 编译的独立可执行文件

实现：
```typescript
return process.versions.bun !== undefined
```

### isInBundledMode

```typescript
export function isInBundledMode(): boolean
```

检测是否作为 Bun 编译的独立可执行文件运行。

独立可执行文件包含嵌入式文件，通过检查 `Bun.embeddedFiles` 判断。

实现：
```typescript
return (
  typeof Bun !== 'undefined' &&
  Array.isArray(Bun.embeddedFiles) &&
  Bun.embeddedFiles.length > 0
)
```

## 设计要点

1. **简单检测**: 基于 Bun 特定的全局变量
2. **无依赖**: 纯原生检测，无需额外模块
3. **独立可执行文件**: 通过嵌入式文件存在性判断

## 与其他文件的关系

该模块无外部依赖，可被任何模块安全导入。

## 注意事项

1. **Bun 全局变量**: 依赖 `process.versions.bun` 和 `Bun.embeddedFiles`
2. **Node 兼容性**: 在 Node.js 环境下返回 false
3. **嵌入文件**: 独立可执行文件必须包含至少一个嵌入式文件
