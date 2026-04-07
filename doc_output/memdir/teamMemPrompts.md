# teamMemPrompts.ts — 团队内存提示词

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/teamMemPrompts.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

构建团队内存和自动内存同时启用时的组合提示词。提供包含 private 和 team 两个目录的完整内存系统指导。

## 核心内容详解

### 主要导出

**buildCombinedMemoryPrompt(extraGuidelines?, skipIndex?)** — 构建组合内存提示词

参数:
- `extraGuidelines`: 额外指导字符串数组 (可选)
- `skipIndex`: 是否跳过索引说明 (可选，默认 false)

返回: 完整的内存提示词字符串

### 提示词结构

1. **标题**:
   - `# Memory`

2. **目录说明**:
   - 说明 private 和 team 两个目录
   - 提供路径信息
   - DIRS_EXIST_GUIDANCE

3. **用途说明**:
   - 建立内存系统的目的
   - 用户明确要求时保存/删除

4. **内存范围**:
   - private: 用户和 Claude 之间私有
   - team: 项目目录内所有用户共享

5. **类型章节** (TYPES_SECTION_COMBINED):
   - 包含 `<scope>` 标签
   - 每种类型说明 private/team/选择指导

6. **不应保存的内容** (WHAT_NOT_TO_SAVE_SECTION):
   - 代码模式、Git 历史等
   - 团队内存中避免敏感数据

7. **保存方法**:
   - 两步骤过程 (skipIndex = false)
   - 单步骤过程 (skipIndex = true)
   - frontmatter 格式
   - MEMORY.md 索引说明

8. **何时访问内存**:
   - 内存相关或用户引用先前工作
   - 明确要求时
   - 忽略内存指令
   - MEMORY_DRIFT_CAVEAT

9. **信任回忆** (TRUSTING_RECALL_SECTION):
   - 验证文件/函数声明
   - 快照是冻结的

10. **内存和其他持久化机制**:
    - 与 Plan 和 Task 的区别

11. **搜索过去上下文**:
    - 使用 buildSearchingPastContextSection

### 保存流程

**标准模式 (skipIndex = false)**:
1. 写入内存文件到相应目录
2. 在相同目录的 MEMORY.md 中添加指针

**跳过索引模式 (skipIndex = true)**:
- 直接写入内存文件到选定目录
- 无前述索引步骤

### 范围指导

每种类型的 `<scope>` 标签:
- **user**: always private
- **feedback**: default to private，项目范围约定时 team
- **project**: private or team，偏向 team
- **reference**: usually team

## 设计要点

1. **组合模式**:
   - 同时处理 private 和 team 目录
   - 用户决定每份内存的范围

2. **安全警告**:
   - 明确警告不在 team 内存中保存敏感数据
   - API 密钥、用户凭证等

3. **特性复用**:
   - 从 memdir.ts 导入共享函数
   - 从 memoryTypes.ts 导入类型章节
   - 避免重复定义

4. **路径注入**:
   - 动态注入 autoDir 和 teamDir 路径
   - 确保路径准确性

5. **索引说明**:
   - 每个目录有自己的 MEMORY.md
   - 条目格式: `- [Title](file.md) — one-line hook`
   - 200 行后截断

## 与其他文件的关系

- **./memdir.ts**: buildSearchingPastContextSection, DIRS_EXIST_GUIDANCE, MAX_ENTRYPOINT_LINES
- **./memoryTypes.ts**: TYPES_SECTION_COMBINED, WHAT_NOT_TO_SAVE_SECTION, 等
- **./paths.ts**: getAutoMemPath
- **./teamMemPaths.ts**: getTeamMemPath

## 注意事项

1. 团队内存需要 auto + team 特性都启用
2. 每个目录有自己的 MEMORY.md 索引
3. 用户根据类型指导选择 private 或 team
4. 敏感数据绝不应保存到 team 内存
5. 提示词使用 XML 标签组织内容
