# reconnection.ts — Swarm 重连模块

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/reconnection.ts`  
**关联模块**: AppState、队友管理  
**主要依赖**: `src/state/AppState.js`, `src/utils/teammate.js`

## 功能概述

本文件处理 teammates 的 Swarm 上下文初始化：
- 新启动：从 CLI 参数初始化（通过 `dynamicTeamContext`）
- 恢复会话：从转录本中存储的 teamName/agentName 初始化

## 核心内容详解

### 核心函数

#### `computeInitialTeamContext()`
计算 AppState 的初始 `teamContext`：
- 在 `main.tsx` 中同步调用，在首次渲染前计算
- 从 `getDynamicTeamContext()` 获取 CLI 参数设置的上下文
- 读取团队文件获取领导 agent ID
- 返回完整的 `TeamContext` 对象或 `undefined`（如果不是 teammate）

返回的上下文包含：
```typescript
{
  teamName: string           // 团队名称
  teamFilePath: string       // 团队文件路径
  leadAgentId: string        // 领导 Agent ID
  selfAgentId: string | null // 自身 Agent ID（Leader 为 null）
  selfAgentName: string      // 自身 Agent 名称
  isLeader: boolean          // 是否为领导
  teammates: {}              // 队友映射（初始为空）
}
```

#### `initializeTeammateContextFromSession(setAppState, teamName, agentName)`
从恢复的会话初始化 teammate 上下文：
- 在恢复具有 teamName/agentName 的会话时调用
- 设置 `teamContext` 使心跳和其他 Swarm 功能正常工作
- 在团队成员中查找以获取 agentId

## 设计要点

1. **同步计算**: `computeInitialTeamContext` 在 React 渲染前同步执行，避免 useEffect 变通方案
2. **团队文件读取**: 使用同步读取（`readTeamFile`）以支持同步初始化
3. **Leader 检测**: `isLeader` 基于 `agentId` 是否存在判断

## 与其他文件的关系

- **main.tsx**: 调用 `computeInitialTeamContext` 初始化状态
- **teamHelpers.ts**: 使用 `readTeamFile` 和 `getTeamFilePath`
- **teammate.ts**: 使用 `getDynamicTeamContext`

## 注意事项

- 团队成员可能已从团队文件中移除（如被删除），此时 agentId 为 undefined
- 恢复会话时 `teammates` 初始为空对象，后续由心跳机制填充
- 日志使用 `logForDebugging` 便于调试 Swarm 问题
