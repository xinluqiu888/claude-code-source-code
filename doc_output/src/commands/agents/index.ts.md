# index.ts — agents 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/agents/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 10 行 |
| 主要职责 | 定义 agents 命令的元数据和懒加载配置 |

## 功能概述

该文件是 `/agents` 命令的入口配置文件，遵循 Claude Code 的命令注册模式。它定义了命令的基本信息（名称、描述），并通过懒加载机制延迟加载实际实现，以减少启动时间。

## 核心内容详解

### 命令配置对象

```typescript
const agents = {
  type: 'local-jsx',              // 命令类型：本地 JSX 组件
  name: 'agents',                 // 命令名称
  description: 'Manage agent configurations',  // 命令描述
  load: () => import('./agents.js'),  // 懒加载实现
} satisfies Command
```

### 配置说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 表示这是一个本地 JSX 组件命令 |
| `name` | `'agents'` | 用户输入的命令名（即 `/agents`） |
| `description` | `'Manage agent configurations'` | 在命令帮助中显示的描述 |
| `load` | `() => import('./agents.js')` | 动态导入实现文件，实现懒加载 |

## 设计要点

1. **懒加载优化**：通过 `load` 函数延迟加载 `agents.tsx` 的实现，减少主线程启动时间
2. **类型安全**：使用 `satisfies Command` 确保配置符合 Command 接口规范
3. **无参数设计**：该命令不需要参数，因此没有 `argumentHint`
4. **单一职责**：仅负责元数据定义，不包含任何业务逻辑

## 与其他文件的关系

- **agents.tsx**: 实际的命令实现文件，包含 UI 组件和业务逻辑
- **commands.ts**: 定义 `Command` 类型和命令注册机制
- **AgentsMenu.tsx**: 代理菜单组件（由 agents.tsx 导入）

## 注意事项

1. **文件扩展名**：`load` 函数中导入的是 `.js` 而非 `.tsx`，这是因为构建工具会处理 TypeScript 转换
2. **懒加载路径**：路径是相对于当前文件的位置
3. **默认导出**：命令对象作为默认导出供命令系统注册使用
4. **无别名**：该命令没有配置别名
