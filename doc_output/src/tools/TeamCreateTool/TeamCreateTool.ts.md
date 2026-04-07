# TeamCreateTool.ts — 团队创建工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TeamCreateTool/TeamCreateTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 创建多智能体协作团队

## 功能概述

TeamCreateTool 用于创建新的团队协作环境，支持多个 Claude Code 智能体协同工作。创建团队时会同时创建对应的任务列表。

## 核心内容详解

### 输入 Schema

```typescript
{
  team_name: string       // 团队名称
  description?: string    // 团队描述/目的
  agent_type?: string     // 团队负责人的类型/角色
}
```

### 输出 Schema

```typescript
{
  team_name: string       // 团队名称
  team_file_path: string  // 团队配置文件路径
  lead_agent_id: string   // 负责人智能体 ID
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `generateUniqueTeamName()` | 生成唯一团队名称（避免重复） |
| `validateInput()` | 验证团队名称不为空 |
| `call()` | 创建团队并初始化相关资源 |

### 核心逻辑

1. **检查现有团队**: 限制负责人只能管理一个团队
2. **生成唯一名称**: 如果名称已存在，自动生成新名称
3. **创建团队文件**: 写入 `~/.claude/teams/{team-name}/config.json`
4. **初始化任务列表**: 重置并创建对应的任务目录
5. **设置团队上下文**: 更新 AppState 的 teamContext
6. **记录分析事件**: 发送团队创建事件

## 设计要点

### 功能开关
- `isAgentSwarmsEnabled()` 控制是否启用

### 唯一性保证
- 如果团队名称已存在，自动生成单词 slug

### 资源关联
- 团队 = 任务列表（1:1 对应关系）
- 团队文件包含成员信息

### 负责人限制
- 一个负责人只能管理一个团队
- 需先删除当前团队才能创建新团队

## 与其他文件的关系

### 依赖
- `../../utils/swarm/teamHelpers.js` — 团队辅助函数
- `../../utils/swarm/teammateLayoutManager.js` — 队友布局管理
- `../../utils/tasks.js` — 任务管理
- `../../services/analytics/index.js` — 分析事件
- `./constants.ts` — 工具名称常量
- `./prompt.ts` — 工具提示信息
- `./UI.tsx` — UI 渲染组件

### 使用场景
- 用户明确要求使用团队/群/swarm
- 复杂任务需要并行工作
