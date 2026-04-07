# prompt.ts — 团队创建工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/TeamCreateTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 TeamCreate 工具的动态提示信息

## 核心内容详解

### 导出函数

```typescript
export function getPrompt(): string
```

### 提示信息结构

#### 使用场景
- 用户明确要求使用团队、swarm 或智能体组
- 用户提到希望智能体一起工作、协调或协作
- 任务足够复杂，需要多个智能体并行工作

#### 选择智能体类型
根据任务需求选择 `subagent_type`:
- **只读智能体** (Explore, Plan): 只能搜索，不能编辑文件
- **全功能智能体**: 拥有所有工具访问权限
- **自定义智能体**: 在 `.claude/agents/` 中定义

#### 创建工作流
1. **创建团队** — TeamCreate
2. **创建任务** — TaskCreate, TaskList 等
3. **生成队友** — Agent 工具，使用 `team_name` 和 `name` 参数
4. **分配任务** — TaskUpdate 的 `owner` 参数
5. **队友工作** — 在分配的任务上工作并通过 TaskUpdate 标记完成
6. **队友空闲** — 每轮后自动空闲并发送通知
7. **关闭团队** — 完成后通过 SendMessage 发送 `shutdown_request`

#### 任务所有权
使用 TaskUpdate 的 `owner` 参数分配任务。

#### 自动消息传递
队友消息自动传递给负责人，无需手动检查收件箱。

#### 队友空闲状态
- 空闲队友可以接收消息
- 空闲通知自动发送
- 不要将空闲视为错误

#### 发现团队成员
通过读取团队配置文件 `~/.claude/teams/{team-name}/config.json` 发现其他成员。

#### 任务列表协调
团队共享任务列表，位于 `~/.claude/tasks/{team-name}/`。

## 与其他文件的关系

### 被依赖
- `TeamCreateTool.ts` — 工具主实现文件
