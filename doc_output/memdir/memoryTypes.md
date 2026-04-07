# memoryTypes.ts — 内存类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/memoryTypes.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

内存类型分类系统，定义了四种类型的内存分类 (user、feedback、project、reference)，并提供用于构建内存提示词的章节模板。

## 核心内容详解

### 内存类型

```typescript
export const MEMORY_TYPES = ['user', 'feedback', 'project', 'reference'] as const
export type MemoryType = (typeof MEMORY_TYPES)[number]
```

### 主要导出

1. **parseMemoryType(raw)** — 解析内存类型
   - 将原始 frontmatter 值解析为 MemoryType
   - 无效或缺失值返回 undefined
   - 向后兼容无 `type:` 字段的旧文件

2. **TYPES_SECTION_COMBINED** — 组合模式类型章节
   - 用于 private + team 目录组合模式
   - 包含 `<scope>` 标签 (private/team)
   - 示例包含 team/private 限定符

3. **TYPES_SECTION_INDIVIDUAL** — 个人模式类型章节
   - 用于单目录个人模式
   - 无 `<scope>` 标签
   - 示例使用简单 `[saves X memory: …]` 格式

4. **WHAT_NOT_TO_SAVE_SECTION** — 不应保存的内容章节
   - 代码模式、架构、文件路径
   - Git 历史、调试解决方案
   - 已记录在 CLAUDE.md 中的内容
   - 临时任务细节
   - 明确的保存门控 (H2)

5. **MEMORY_DRIFT_CAVEAT** — 内存漂移警告
   - 关于内存可能过时的警告
   - 建议验证内存与当前状态

6. **WHEN_TO_ACCESS_SECTION** — 何时访问内存章节
   - 内存相关或用户引用先前工作
   - 用户明确要求时
   - 忽略内存指令处理
   - 包含 MEMORY_DRIFT_CAVEAT

7. **TRUSTING_RECALL_SECTION** — 信任回忆章节
   - 验证文件存在性
   - 搜索函数或标志
   - 活动日志和架构快照是冻结的

8. **MEMORY_FRONTMATTER_EXAMPLE** — frontmatter 格式示例
   - name: 内存名称
   - description: 单行描述
   - type: user/feedback/project/reference
   - 内容结构指导

## 内存类型详情

### user 类型
- **范围**: 总是 private
- **描述**: 用户信息、角色、目标、责任、知识
- **保存时机**: 了解用户角色、偏好、责任或知识时
- **使用方式**: 根据用户画像调整协作方式

### feedback 类型
- **范围**: 默认 private，项目范围约定时 team
- **描述**: 用户关于工作方式的指导
- **保存时机**: 纠正或确认非明显方法时
- **使用方式**: 避免重复接收相同指导
- **结构**: 规则 + Why + How to apply

### project 类型
- **范围**: private 或 team，偏向 team
- **描述**: 项目上下文、目标、计划、bug、事件
- **保存时机**: 了解谁、做什么、何时、为何时
- **使用方式**: 理解请求背景，做出更好建议
- **结构**: 事实 + Why + How to apply

### reference 类型
- **范围**: 通常是 team
- **描述**: 外部系统信息指针
- **保存时机**: 了解外部资源及其用途时
- **使用方式**: 用户引用外部系统时

## 设计要点

1. **故意重复**:
   - TYPES_SECTION_* 导出故意重复而非生成
   - 便于每种模式的独立编辑

2. **Eval 验证**:
   - 多个章节经过 eval 验证
   - H1 (验证函数/文件声明)
   - H5 (读取端噪声拒绝)
   - H6 (分支污染)

3. **标记预算**:
   - 合并旧项目以减少标记
   - 约 70 tokens 的旧版 → 约 73 tokens 的新版

4. **XML 格式**:
   - 使用 XML 标签组织内容
   - `<types>`, `<type>`, `<name>`, `<scope>`, `<description>` 等

5. **向后兼容**:
   - 无 type 字段的旧文件继续工作
   - 未知类型优雅降级

## 与其他文件的关系

- **./memdir.ts**: 使用章节构建提示词
- **./teamMemPrompts.ts**: 使用 TYPES_SECTION_COMBINED

## 注意事项

1. 四种类型捕获无法从当前项目状态推导的上下文
2. 代码模式、架构、Git 历史应排除 (可推导)
3. 相对日期应转换为绝对日期
4. 反馈类型应记录成功和失败
5. 用户明确要求时立即保存
