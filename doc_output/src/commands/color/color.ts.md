# color.ts — 会话颜色设置命令

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/color/color.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 93 行 |
| 主要职责 | 实现会话颜色设置功能，支持预设颜色和重置 |

## 功能概述

该文件实现了 `/color` 命令，允许用户设置当前会话的提示栏颜色。支持多种预设颜色，也支持重置为默认颜色。颜色设置会持久化到会话存储中，并在 AppState 中立即生效。

## 核心内容详解

### 重置别名

```typescript
const RESET_ALIASES = ['default', 'reset', 'none', 'gray', 'grey'] as const
```

### 主要函数

**call 函数**
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: ToolUseContext & LocalJSXCommandContext,
  args: string,
): Promise<null>
```

### 功能流程

1. **队友检查**：如果是队友会话（`isTeammate()`），禁止设置颜色
2. **空参数处理**：显示可用颜色列表
3. **重置处理**：匹配 `RESET_ALIASES` 时重置为默认颜色
4. **颜色验证**：检查颜色是否在 `AGENT_COLORS` 列表中
5. **持久化**：调用 `saveAgentColor` 保存到会话存储
6. **状态更新**：更新 `AppState.standaloneAgentContext.color`

### 支持的颜色

颜色列表由 `AGENT_COLORS` 常量定义（在 `agentColorManager.ts` 中），包括多种预设颜色名称。

## 设计要点

1. **队友保护**：队友的颜色由团队领导者分配，不能自行设置
2. **立即生效**：更新 AppState 后颜色立即在UI中体现
3. **持久化**：颜色设置跨会话保持
4. **类型安全**：使用 TypeScript 常量确保颜色值有效

## 与其他文件的关系

- **agentColorManager.ts**: 提供颜色列表和类型定义
- **sessionStorage.ts**: 提供颜色持久化功能
- **teammate.ts**: 提供队友检测功能
- **index.ts**: 命令注册
