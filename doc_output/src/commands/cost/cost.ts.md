# cost.ts — 会话成本统计命令

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/cost/cost.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 24 行 |
| 主要职责 | 显示当前会话的总成本和持续时间 |

## 功能概述

该文件实现了 `/cost` 命令，用于显示当前会话的成本统计和持续时间。对于订阅用户会显示订阅状态信息，对于 Ant 员工还会显示详细的成本细分。

## 核心内容详解

### 主要函数

**call 函数**
```typescript
export const call: LocalCommandCall = async () => {
  if (isClaudeAISubscriber()) {
    // 订阅用户：显示订阅状态
    let value: string
    if (currentLimits.isUsingOverage) {
      value = 'You are currently using your overages...'
    } else {
      value = 'You are currently using your subscription...'
    }
    // Ant 员工额外显示成本
    if (process.env.USER_TYPE === 'ant') {
      value += `\n\n[ANT-ONLY] Showing cost anyway:\n ${formatTotalCost()}`
    }
    return { type: 'text', value }
  }
  // 非订阅用户：直接显示成本
  return { type: 'text', value: formatTotalCost() }
}
```

### 输出内容

**订阅用户**：
- 当前使用订阅还是超额额度
- 自动切换说明

**Ant 员工**（额外）：
- 详细的成本统计

**非订阅用户**：
- 当前会话的成本统计

## 设计要点

1. **用户类型感知**：根据用户类型显示不同信息
2. **订阅状态**：区分订阅额度和超额使用
3. **Ant 专用**：为内部员工显示额外成本数据

## 与其他文件的关系

- **cost-tracker.ts**: 提供 `formatTotalCost` 函数
- **claudeAiLimits.ts**: 提供 `currentLimits` 对象
- **auth.ts**: 提供 `isClaudeAISubscriber` 函数
- **index.ts**: 命令注册
