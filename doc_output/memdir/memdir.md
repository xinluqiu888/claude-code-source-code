# memdir.ts — 内存目录主模块

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/memdir.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

Claude Code 的内存系统主模块，管理持久化文件内存的加载、构建和提示词集成。支持自动内存 (auto memory)、团队内存 (team memory) 和助手模式 (KAIROS) 的每日日志功能。

## 核心内容详解

### 常量定义

```typescript
export const ENTRYPOINT_NAME = 'MEMORY.md'
export const MAX_ENTRYPOINT_LINES = 200
export const MAX_ENTRYPOINT_BYTES = 25_000  // ~125 chars/line
const AUTO_MEM_DISPLAY_NAME = 'auto memory'
```

### 主要导出

1. **truncateEntrypointContent(raw)** — 截断入口点内容
   - 同时应用行数和字节数限制
   - 先按行截断，再按字节截断
   - 返回截断结果和统计信息

2. **DIR_EXISTS_GUIDANCE / DIRS_EXIST_GUIDANCE** — 目录存在指导
   - 提示模型目录已存在，可直接写入
   - 避免模型浪费时间检查目录存在性

3. **ensureMemoryDirExists(memoryDir)** — 确保内存目录存在
   - 幂等操作
   - 使用 FsOperations.mkdir 递归创建
   - 静默处理 EEXIST 错误

4. **buildMemoryLines()** — 构建内存提示词行
   - 构建类型化内存行为指导
   - 支持个人模式 (无 scope 标签)
   - 支持跳过索引模式

5. **buildMemoryPrompt()** — 构建带内容的内存提示词
   - 用于工作流内存 (无 getClaudeMds 等效项)
   - 读取并包含 MEMORY.md 内容
   - 记录内存目录统计

6. **buildAssistantDailyLogPrompt()** — 助手每日日志提示词
   - KAIROS 特性下的每日日志模式
   - 仅追加模式写入日期命名日志文件
   - 夜间进程将日志提炼为 MEMORY.md

7. **buildSearchingPastContextSection()** — 构建搜索过去上下文章节
   - 特性门控 (tengu_coral_fern)
   - 提供内存目录和会话日志搜索指导

8. **loadMemoryPrompt()** — 加载统一内存提示词
   - 根据启用的内存系统分发
   - 支持 auto + team 组合模式
   - 支持 KAIROS 每日日志模式

### 内存类型系统

基于四种类型的分类体系：
- **user**: 用户信息、角色、偏好
- **feedback**: 用户反馈、指导
- **project**: 项目上下文、工作进展
- **reference**: 外部系统引用

### 特性门控

- **TEAMMEM**: 团队内存支持
- **KAIROS**: 助手模式和每日日志
- **tengu_moth_copse**: 跳过索引模式
- **tengu_coral_fern**: 搜索过去上下文

## 设计要点

1. **目录预创建**:
   - 加载提示词时确保目录存在
   - 模型可直接写入无需检查

2. **截断策略**:
   - 行数限制优先 (自然边界)
   - 字节数限制在后 (最后换行处截断)
   - 防止长行索引溢出

3. **日志记录**:
   - 异步记录内存目录统计
   - Fire-and-forget 不阻塞提示词构建

4. **模式分发**:
   - 根据特性门控动态选择提示词模式
   - 支持多种内存系统的组合

5. **Cowork 支持**:
   - 通过环境变量注入额外指导
   - 适用于所有构建器

## 与其他文件的关系

- **./paths.ts**: 内存路径获取函数
- **./memoryTypes.ts**: 内存类型定义和章节
- **./teamMemPrompts.ts**: 团队内存提示词
- **./teamMemPaths.ts**: 团队内存路径

## 注意事项

1. MEMORY.md 是索引而非内存，条目应为一行
2. 200 行后的内容将被截断
3. 助手模式的每日日志是仅追加的
4. 团队内存需要自动内存启用
5. 便签本目录需要 tengu_scratch 特性
