# index.ts — remote-control (bridge) 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/bridge/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 26 行 |
| 主要职责 | 定义 remote-control (rc) 命令的元数据、启用条件和懒加载配置 |

## 功能概述

该文件是 `/remote-control` 命令（别名 `/rc`）的入口配置文件。与其他命令不同，该命令具有动态启用逻辑，只有在特性标志启用且桥接功能已启用时才可用。这体现了 Claude Code 中命令的条件可见性模式。

## 核心内容详解

### 启用检查函数

```typescript
function isEnabled(): boolean {
  if (!feature('BRIDGE_MODE')) {
    return false
  }
  return isBridgeEnabled()
}
```

启用条件：
1. `BRIDGE_MODE` 特性标志必须启用
2. `isBridgeEnabled()` 返回 true（可能涉及运行时配置检查）

### 命令配置对象

```typescript
const bridge = {
  type: 'local-jsx',
  name: 'remote-control',
  aliases: ['rc'],
  description: 'Connect this terminal for remote-control sessions',
  argumentHint: '[name]',
  isEnabled,
  get isHidden() {
    return !isEnabled()
  },
  immediate: true,
  load: () => import('./bridge.js'),
} satisfies Command
```

### 配置说明

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件命令 |
| `name` | `'remote-control'` | 主命令名 |
| `aliases` | `['rc']` | 别名 `/rc` |
| `description` | `'Connect this terminal for remote-control sessions'` | 功能描述 |
| `argumentHint` | `'[name]'` | 可选的会话名称参数 |
| `isEnabled` | `isEnabled` | 动态启用检查函数 |
| `isHidden` | getter | 未启用时隐藏命令 |
| `immediate` | `true` | 立即执行模式 |
| `load` | 函数 | 懒加载实现 |

## 设计要点

1. **条件启用**：使用 `isEnabled` 函数动态控制命令可用性
2. **动态隐藏**：`isHidden` getter 在未启用时隐藏命令，避免用户困惑
3. **即时执行**：`immediate: true` 表示命令不进入主消息循环
4. **双重检查**：同时检查特性标志和运行时启用状态

## 与其他文件的关系

- **bridge.tsx**: 实际的命令实现文件
- **bridgeEnabled.ts**: 提供 `isBridgeEnabled()` 运行时检查
- **commands.ts**: 定义 `Command` 类型和注册机制

## 注意事项

1. **懒加载路径**：导入路径使用 `.js` 扩展名
2. **即时命令**：作为即时命令，它可能绕过某些常规处理
3. **特性标志依赖**：需要 `BRIDGE_MODE` 特性才能启用
4. **可选参数**：`[name]` 表示会话名称是可选的
