# toolPermission/handlers/ — 权限处理器目录

> **一句话总结**：不同运行模式下的工具权限处理实现

---

## 目录职责

本目录包含三种不同运行模式下的权限处理实现：
1. **交互式处理器**: 主代理（Main Agent）使用，显示对话框
2. **协调器处理器**: 协调器工作器使用，顺序等待自动化检查
3. **Swarm Worker 处理器**: 队友工作器使用，转发给 Leader 决策

## 文件清单

| 文件 | 职责简述 |
|------|---------|
| [interactiveHandler.ts](./interactiveHandler.ts.md) | 交互式权限处理，支持用户对话框、CCR Bridge、MCP 通道、分类器和钩子 |
| [coordinatorHandler.ts](./coordinatorHandler.ts.md) | 协调器权限处理，顺序等待钩子和分类器检查 |
| [swarmWorkerHandler.ts](./swarmWorkerHandler.ts.md) | Swarm Worker 权限处理，转发请求给 Leader 并等待响应 |

## 处理器对比

| 特性 | interactiveHandler | coordinatorHandler | swarmWorkerHandler |
|------|-------------------|-------------------|-------------------|
| 适用角色 | 主代理 | 协调器工作器 | 队友工作器 |
| 用户交互 | 是（对话框） | 否 | 否 |
| 自动化检查 | 并行竞态 | 顺序等待 | 仅分类器 |
| 转发给 Leader | 否 | 否 | 是 |
| CCR Bridge | 支持 | 否 | 否 |
| MCP 通道 | 支持 | 否 | 否 |

## 内部协作关系

```
PermissionContext.ts
    ↓ 提供 PermissionContext
    ↓ 调用
interactiveHandler.ts → 用户/CRC/通道/分类器/钩子（并行竞态）
coordinatorHandler.ts → 钩子 → 分类器（顺序）
swarmWorkerHandler.ts → 分类器 → Leader（转发）
```

## 核心机制

### 竞态模式 (interactiveHandler)

多个权限来源（用户、Bridge、通道、分类器、钩子）并行竞争，第一个响应者获胜。

使用 `createResolveOnce` 的 `claim()` 方法保证原子性：
```typescript
const { resolve: resolveOnce, claim } = createResolveOnce(resolve)

// 每个回调中
if (!claim()) return // 已被其他来源处理
// 安全处理
```

### 转发模式 (swarmWorkerHandler)

工作器不直接决策，而是通过 Mailbox 转发给 Leader：
1. 创建权限请求
2. 注册回调（先于发送）
3. 通过 Mailbox 发送
4. 等待 Leader 响应
5. 设置等待指示器

### 顺序模式 (coordinatorHandler)

按顺序等待自动化检查，如果都未解决则返回 null：
1. 尝试钩子
2. 尝试分类器（如果启用）
3. 返回决策或 null
