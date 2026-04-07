# prompt.ts — 压缩提示词模板

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/prompt.ts`
- **作用域**: 压缩摘要生成的提示词模板
- **主要导出**:
  - `getCompactPrompt`: 获取完整压缩提示词
  - `getPartialCompactPrompt`: 获取部分压缩提示词
  - `formatCompactSummary`: 格式化压缩摘要
  - `getCompactUserSummaryMessage`: 获取用户摘要消息

## 功能概述

提供用于生成对话摘要的提示词模板。包括完整对话压缩、部分压缩（from/up_to 方向）的提示词，以及摘要格式化功能。

## 核心内容详解

### 提示词结构

#### NO_TOOLS_PREAMBLE
强制纯文本响应的前置说明：
```
CRITICAL: Respond with TEXT ONLY. Do NOT call any tools.
- Do NOT use Read, Bash, Grep, Glob, Edit, Write, or ANY other tool.
```

#### 完整压缩提示词 (BASE_COMPACT_PROMPT)
包含以下章节：
1. Primary Request and Intent: 用户的主要请求和意图
2. Key Technical Concepts: 关键技术概念
3. Files and Code Sections: 文件和代码段
4. Errors and fixes: 错误及修复方法
5. Problem Solving: 问题解决记录
6. All user messages: 所有非工具结果的用户消息
7. Pending Tasks: 待处理任务
8. Current Work: 当前工作内容
9. Optional Next Step: 可选的下一步

#### 部分压缩提示词 (PARTIAL_COMPACT_PROMPT)
针对最近消息的压缩，结构与完整版类似，但聚焦于最近的消息。

#### 'up_to' 方向提示词 (PARTIAL_COMPACT_UP_TO_PROMPT)
用于前缀保留的部分压缩，摘要将放在会话开始位置。

### 主要函数

#### `getCompactPrompt(customInstructions?)`
返回完整压缩提示词，支持附加自定义说明。

#### `getPartialCompactPrompt(customInstructions?, direction?)`
返回部分压缩提示词：
- `direction: 'from'`: 从某点开始向后摘要
- `direction: 'up_to'`: 摘要到某点为止

#### `formatCompactSummary(summary)`
格式化原始摘要：
1. 移除 `<analysis>` 草稿块
2. 将 `<summary>` 标签替换为可读标题
3. 清理多余空白

#### `getCompactUserSummaryMessage(summary, suppressFollowUpQuestions?, transcriptPath?, recentMessagesPreserved?)`
构建显示给用户的摘要消息：
- 包含格式化后的摘要
- 可选：添加完整转录路径
- 可选：指示是否保留了最近消息
- 可选：抑制后续问题的提示

## 设计要点

1. **无工具模式**: 通过前置说明禁止工具调用
2. **分析草稿**: 使用 `<analysis>` 标签作为模型思考空间
3. **结构化输出**: 使用 `<summary>` 标签包裹最终摘要
4. **方向支持**: 支持 'from' 和 'up_to' 两种部分压缩方向
5. **自主模式感知**: 检测 PROACTIVE/KAIROS 模式并添加相应说明

## 与其他文件的关系

- **compact.ts**: 使用这些提示词生成摘要
- **proactive/index.ts**: 检测主动模式状态

## 注意事项

1. **工具禁用**: 必须确保压缩代理不调用工具（maxTurns: 1）
2. **格式解析**: `formatCompactSummary` 依赖 XML 标签格式
3. **自定义说明**: 支持用户提供的自定义压缩说明
