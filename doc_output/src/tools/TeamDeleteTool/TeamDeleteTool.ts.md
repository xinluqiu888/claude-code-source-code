# TeamDeleteTool.ts — 团队删除工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TeamDeleteTool/TeamDeleteTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 清理团队和相关任务目录

## 功能概述

TeamDeleteTool 用于在工作完成后清理团队资源。它会删除团队目录和任务目录，并清除团队上下文。

## 核心内容详解

### 输入 Schema

```typescript
{}  // 无需输入参数，自动使用当前会话的团队上下文
```

### 输出 Schema

```typescript
{
  success: boolean
  message: string
  team_name?: string
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `call()` | 执行团队清理操作 |

### 核心逻辑

1. **检查活跃成员**:
   - 读取团队配置文件
   - 过滤出非负责人成员
   - 检查是否有 `isActive !== false` 的成员

2. **安全保护**:
   - 如果有活跃成员，返回错误
   - 提示先使用 `requestShutdown` 终止队友

3. **执行清理**:
   - 调用 `cleanupTeamDirectories()` 删除目录
   - 取消注册团队清理
   - 清除队友颜色分配
   - 清除负责人团队名称

4. **状态更新**:
   - 清除 AppState 中的 teamContext
   - 清空收件箱消息队列

5. **分析记录**:
   - 发送团队删除事件

## 设计要点

### 功能开关
- `isAgentSwarmsEnabled()` 控制是否启用

### 安全机制
- 拒绝删除有活跃成员的团队
- 必须先优雅终止所有队友

### 自动发现
- 自动从 AppState 获取当前团队名称
- 无需用户输入团队名称

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
- 所有队友完成工作后清理资源
- 团队会话结束时的清理
