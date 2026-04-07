# prompt.ts — 团队删除工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TeamDeleteTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TeamDelete 工具的提示信息

## 核心内容详解

### 导出函数

```typescript
export function getPrompt(): string
```

### 提示信息结构

#### 操作说明
移除团队和工作目录当 swarm 工作完成时。

#### 执行操作
- 删除团队目录 (`~/.claude/teams/{team-name}/`)
- 删除任务目录 (`~/.claude/tasks/{team-name}/`)
- 清除当前会话的团队上下文

#### 重要提示
TeamDelete 在团队仍有活跃成员时会失败。需要先优雅终止队友，然后等所有队友关闭后再调用 TeamDelete。

#### 使用时机
在所有队友完成工作且希望清理团队资源时使用。

团队名称自动从当前会话的团队上下文中确定。

## 与其他文件的关系

### 被依赖
- `TeamDeleteTool.ts` — 工具主实现文件
