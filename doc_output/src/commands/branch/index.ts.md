# index.ts — branch 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/branch/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 14 行 |
| 主要职责 | 定义 branch/fork 命令的元数据、别名和懒加载配置 |

## 功能概述

该文件是 `/branch` 命令的入口配置文件。特别的是，它支持条件别名——当 `FORK_SUBAGENT` 特性未启用时，`fork` 作为 `branch` 的别名可用。这体现了 Claude Code 中命令别名的动态配置模式。

## 核心内容详解

### 命令配置对象

```typescript
const branch = {
  type: 'local-jsx',
  name: 'branch',
  // 'fork' alias only when /fork doesn't exist as its own command
  aliases: feature('FORK_SUBAGENT') ? [] : ['fork'],
  description: 'Create a branch of the current conversation at this point',
  argumentHint: '[name]',
  load: () => import('./branch.js'),
} satisfies Command
```

### 配置说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件命令 |
| `name` | `'branch'` | 主命令名 |
| `aliases` | 条件数组 | `FORK_SUBAGENT` 特性启用时为 `[]`，否则为 `['fork']` |
| `description` | `'Create a branch of the current conversation at this point'` | 功能描述 |
| `argumentHint` | `'[name]'` | 可选的分支名称参数 |
| `load` | 函数 | 懒加载实现 |

### 条件别名说明

```typescript
// 'fork' alias only when /fork doesn't exist as its own command
aliases: feature('FORK_SUBAGENT') ? [] : ['fork'],
```

这个条件别名的设计意图：
- 当 `FORK_SUBAGENT` 特性启用时，存在独立的 `/fork` 命令，此时 `branch` 不需要 `fork` 别名
- 当特性未启用时，`fork` 作为 `branch` 的别名，用户可以使用 `/fork` 或 `/branch`

## 设计要点

1. **特性驱动别名**：使用特性标志动态控制别名可用性
2. **向后兼容**：即使 `/fork` 作为独立命令存在， `/branch` 仍然可用
3. **条件编译**：通过特性标志实现功能的渐进式发布
4. **统一接口**：提供一致的命令接口，底层实现可以变化

## 与其他文件的关系

- **branch.ts**: 实际的命令实现文件，包含分支创建逻辑
- **commands.ts**: 定义 `Command` 类型和注册机制
- **bun:bundle**: 提供 `feature()` 函数用于特性标志检查

## 注意事项

1. **懒加载路径**：导入路径使用 `.js` 扩展名
2. **可选参数**：`[name]` 表示分支名称是可选的
3. **动态别名**：别名数组在运行时根据特性标志确定
4. **语义清晰**：`branch` 和 `fork` 在功能上等价，但 `fork` 更具 Git 语义
