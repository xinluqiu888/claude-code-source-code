# SkillTool.ts — 技能执行工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SkillTool/SkillTool.ts`
- **作用**: 在主线对话或forked子代理中执行技能（slash command）

## 功能概述

该工具执行指定的技能（slash command）。支持inline执行（在主线对话中展开提示）和forked执行（在独立子代理中运行）。支持本地技能、MCP技能和远程技能（实验性）。

## 核心内容详解

### 输入Schema

```typescript
z.object({
  skill: z.string().describe('The skill name. E.g., "commit", "review-pr", or "pdf"'),
  args: z.string().optional().describe('Optional arguments for the skill'),
})
```

### 输出Schema

支持union类型：

**Inline输出**:
```typescript
{
  success: boolean,
  commandName: string,
  allowedTools?: string[],
  model?: string,
  status: 'inline'
}
```

**Forked输出**:
```typescript
{
  success: boolean,
  commandName: string,
  status: 'forked',
  agentId: string,
  result: string
}
```

### 验证逻辑

1. 技能格式非空
2. 去除前导斜杠（如果存在）
3. 检查技能存在
4. 检查非`disableModelInvocation`
5. 检查是prompt类型命令
6. 实验性：处理`_canonical_<slug>`远程技能

### 权限检查

1. 检查deny规则（按内容和前缀匹配）
2. 实验性：远程canonical技能自动允许（ant-only）
3. 检查allow规则
4. 仅使用safe properties的技能自动允许
5. 默认询问用户

Safe properties白名单包含：type, progressMessage, contentLength, argNames, model, effort, source, pluginInfo等。

### 执行逻辑

#### Inline执行

1. 处理slash command（`processPromptSlashCommand`）
2. 提取metadata（allowedTools, model, effort）
3. 记录遥测
4. 过滤并标记消息（`tagMessagesWithToolUseID`）
5. 返回newMessages和contextModifier

#### Forked执行

1. 准备forked上下文（`prepareForkedCommandContext`）
2. 运行子代理（`runAgent`）
3. 收集消息并报告进度
4. 提取结果文本
5. 清理invokedSkills状态

#### 远程技能执行（实验性）

1. 验证slug已发现
2. 加载远程技能（`loadRemoteSkill`）
3. 记录遥测
4. 剥离YAML frontmatter
5. 添加base directory header
6. 替换`$CLAUDE_SKILL_DIR`和`$CLAUDE_SESSION_ID`
7. 注册并注入为meta user message

## 设计要点

1. **双模式执行**: 支持inline（快速）和forked（隔离）两种执行模式
2. **权限控制**: 支持deny/allow规则和safe properties自动允许
3. **遥测**: 详细的技能使用遥测，包括来源、插件信息等
4. **远程技能**: 实验性支持从AKI/GCS加载远程技能
5. **上下文修改**: inline执行返回contextModifier更新工具权限和模型
6. **进度报告**: forked执行支持进度回调

## 与其他文件的关系

- **commands.ts**: 技能查找和处理
- **forkedAgent.ts**: forked执行准备和结果提取
- **runAgent.ts**: 子代理运行
- **remoteSkillLoader.ts**: 远程技能加载（实验性）
- **prompt.ts**: 工具提示文本
- **UI.tsx**: 渲染函数

## 注意事项

- 一次只能运行一个技能
- 技能-coach需要技能名称避免误报建议
- 远程技能是实验性功能（ant-only）
- forked执行有独立的token预算
