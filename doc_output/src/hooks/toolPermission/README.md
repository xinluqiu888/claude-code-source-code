# toolPermission/ — 工具权限管理目录

> **一句话总结**：管理 Claude Code 工具使用权限的上下文、日志和处理器

---

## 目录职责

本目录包含工具权限系统的核心实现，处理：
1. 权限上下文创建和管理
2. 权限决策的日志记录和分析
3. 不同运行模式下的权限处理流程

## 文件清单

| 文件 | 职责简述 |
|------|---------|
| [PermissionContext.ts](./PermissionContext.ts.md) | 创建权限上下文，支持决策日志、持久化、分类器、钩子和队列操作 |
| [permissionLogging.ts](./permissionLogging.md) | 权限决策日志记录，分发到 Statsig、OTel 和代码编辑计数器 |

## 子目录

| 目录 | 职责简述 |
|------|---------|
| [handlers/](./handlers/README.md) | 不同运行模式的权限处理器 |

## 内部协作关系

```
PermissionContext.ts
    ↓ 创建 PermissionContext
    ↓ 使用
permissionLogging.ts (logPermissionDecision)
    ↓ 调用
handlers/interactiveHandler.ts (交互式处理)
handlers/coordinatorHandler.ts (协调器处理)
handlers/swarmWorkerHandler.ts (Swarm Worker 处理)
```

## 核心概念

### 权限上下文 (PermissionContext)

封装了权限检查所需的所有信息和操作：
- 工具信息和输入
- 决策日志记录
- 权限持久化
- 分类器检查
- 钩子执行
- 队列操作

### 决策来源

支持多种决策来源追踪：
- `config`: 配置文件自动批准/拒绝
- `hook`: 权限请求钩子
- `user`: 用户手动选择
- `classifier`: AI 分类器自动审批

### 三种处理模式

1. **交互式** (`interactiveHandler.ts`): 主代理，显示对话框给用户
2. **协调器** (`coordinatorHandler.ts`): 协调器工作器，顺序等待自动化检查
3. **Swarm Worker** (`swarmWorkerHandler.ts`): 队友工作器，转发给 Leader 决策
