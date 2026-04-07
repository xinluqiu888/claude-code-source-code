# prompts.ts — 记忆提取提示词模板

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/extractMemories/prompts.ts`
- **作用域**: 记忆提取代理的提示词构建
- **主要导出**:
  - `buildExtractAutoOnlyPrompt`: 自动记忆提取提示词
  - `buildExtractCombinedPrompt`: 组合自动+团队记忆提取提示词

## 功能概述

为记忆提取子代理构建提示词。提取代理作为主对话的完美 fork 运行——相同的系统提示，相同的消息前缀。主代理的系统提示始终包含完整的保存说明；当主代理自行写入记忆时，`extractMemories.ts` 会跳过该轮次。

## 核心内容详解

### 提示词开头 (opener)

```typescript
function opener(newMessageCount: number, existingMemories: string): string
```

包含：
- 角色定义：记忆提取子代理
- 可用工具：Read、Grep、Glob、只读 Bash、Edit/Write（仅记忆目录）
- 回合预算提示：高效策略（并行读取 → 并行写入）
- 内容限制：仅使用最近消息内容，禁止进一步调查

### 自动记忆提示词

#### `buildExtractAutoOnlyPrompt(newMessageCount, existingMemories, skipIndex?)`

构建仅自动记忆的提取提示词：

1. **开场白**: 角色和工具说明
2. **记忆类型**: 四种类型分类法（TYPES_SECTION_INDIVIDUAL）
3. **不保存内容**: 明确不应保存的内容（WHAT_NOT_TO_SAVE_SECTION）
4. **保存方法**:
   - `skipIndex=false`: 两步流程（写入文件 + 更新 MEMORY.md 索引）
   - `skipIndex=true`: 单步流程（仅写入文件）

### 组合记忆提示词

#### `buildExtractCombinedPrompt(newMessageCount, existingMemories, skipIndex?)`

构建自动+团队记忆的提取提示词（需要 TEAMMEM 特性）：

- 与自动记忆版本类似
- 使用 TYPES_SECTION_COMBINED（包含范围指导）
- 添加团队记忆安全警告：禁止保存敏感数据（API 密钥、用户凭证等）
- 根据类型指导选择目录（private 或 team）

### 记忆类型说明

提示词包含四种记忆类型的详细说明：
1. **Preferences**: 用户偏好（范围：private）
2. **Facts**: 项目事实（范围：团队）
3. **Instructions**: 指令（范围：团队）
4. **Feedback**: 反馈（范围：private）

### 保存指南

#### 标准流程（非 skipIndex）

**步骤 1**: 写入记忆文件
```markdown
---
type: <type>
scope: <scope>
---

记忆内容...
```

**步骤 2**: 更新 MEMORY.md 索引
```markdown
- [Title](file.md) — one-line hook
```

#### Skip Index 流程

直接写入记忆文件，跳过索引更新。

### 前置事项检查

提示词包含现有记忆文件清单，要求代理检查列表避免重复创建。

## 设计要点

1. **双模式支持**: 自动记忆和团队记忆两种模式
2. **索引可选**: 支持跳过索引更新的模式
3. **范围指导**: 组合提示词包含每个类型的范围指导
4. **安全警告**: 明确禁止在共享团队记忆中保存敏感数据
5. **高效策略**: 提示并行读取和写入以节省回合

## 与其他文件的关系

- **memoryTypes.ts**: 提供 MEMORY_FRONTMATTER_EXAMPLE、TYPES_SECTION 等常量
- **extractMemories.ts**: 调用这些函数构建提示词

## 注意事项

1. **TEAMMEM 特性**: 组合提示词需要 `feature('TEAMMEM')` 为 true
2. **目录限制**: Write/Edit 仅允许在记忆目录内
3. **工具限制**: Bash 仅允许只读命令
4. **索引长度**: MEMORY.md 超过 200 行会被截断
