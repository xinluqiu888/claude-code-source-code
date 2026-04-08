# DreamTask.ts — 自动梦境(记忆整合)后台任务UI封装

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/DreamTask/DreamTask.ts`
- **类型**: TypeScript 模块
- **主要功能**: 为自动梦境(memory consolidation)子代理提供任务注册和状态管理

## 功能概述

本文件实现了梦境任务的注册、状态更新和生命周期管理。梦境任务是一个后台运行的自动记忆整合子代理，通过UI表面化(footer pill和Shift+Down对话框)使其可见，而梦境代理本身保持不变。

## 核心内容详解

### 类型定义

#### `DreamTurn`
表示梦境代理的单个助手回合，工具使用被折叠为计数：
```typescript
type DreamTurn = {
  text: string           // 助手文本响应
  toolUseCount: number   // 工具使用次数
}
```

#### `DreamPhase`
梦境阶段，从'starting'到'updating'：
```typescript
type DreamPhase = 'starting' | 'updating'
```

#### `DreamTaskState`
完整的梦境任务状态：
- `type`: 'dream' - 任务类型标识
- `phase`: DreamPhase - 当前阶段
- `sessionsReviewing`: number - 正在审查的会话数
- `filesTouched`: string[] - 通过Edit/Write工具观察到的路径(不完整)
- `turns`: DreamTurn[] - 助手文本响应历史(不包括prompt)
- `abortController`: AbortController - 用于取消任务
- `priorMtime`: number - 用于kill时回滚锁的修改时间

### 主要函数

#### `registerDreamTask`
注册新的梦境任务：
- 生成唯一任务ID
- 创建初始任务状态(phase为'starting')
- 注册到任务框架
- 返回任务ID

#### `addDreamTurn`
添加新的对话回合：
- 更新filesTouched(新路径)
- 当首次出现Edit/Write工具使用时，phase转为'updating'
- 限制最多保留MAX_TURNS(30)个回合
- 跳过空更新(无文本、无工具、无新文件)

#### `completeDreamTask`
标记任务完成：
- 设置状态为'completed'
- 设置endTime和notified标志
- 清除abortController

#### `failDreamTask`
标记任务失败：
- 设置状态为'failed'
- 设置endTime和notified标志

#### `DreamTask.kill`
终止梦境任务：
- 调用abortController.abort()取消任务
- 回滚consolidation锁的mtime(通过rollbackConsolidationLock)
- 设置状态为'killed'

## 设计要点

1. **UI表面化**: 梦境代理本身是无形的forked代理，此文件通过任务注册使其在UI中可见
2. **不完整路径追踪**: filesTouched仅捕获pattern-match的工具调用，bash介导的写入会遗漏
3. **轻量级历史**: 仅保留最近30个回合，避免内存膨胀
4. **锁管理**: 使用priorMtime支持失败后的重试机制

## 与其他文件的关系

- **导入**:
  - `rollbackConsolidationLock` from autoDream服务 - 用于失败时回滚
  - `Task`, `TaskStateBase` from Task - 基础任务接口
  - `registerTask`, `updateTaskState` from task/framework - 任务框架操作

- **被调用者**:
  - autoDream.ts - 创建和管理梦境任务

## 注意事项

1. **filesTouched的局限性**: 文档明确说明这是不完整反映，只能作为"至少这些被触及"的参考
2. **Prompt不包含**: turns数组不包含prompt，仅包含助手响应
3. **无阶段解析**: 虽然梦境prompt有4阶段结构(orient/gather/consolidate/prune)，但不解析它们
4. **通知处理**: 梦境没有面向模型的通知路径，完成通知通过inline appendSystemMessage实现
