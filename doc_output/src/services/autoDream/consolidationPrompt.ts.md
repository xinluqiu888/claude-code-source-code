# consolidationPrompt.ts — 整合提示构建器

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/autoDream/consolidationPrompt.ts`
- **作用域**: 构建记忆整合的提示词
- **主要导出**:
  - `buildConsolidationPrompt`: 构建整合提示词

## 功能概述

从 `dream.ts` 中提取，使自动梦境可以独立于 KAIROS 特性标志运行（dream.ts 受特性门控）。构建指导 Claude 执行记忆整合（"梦境"）的完整提示词。

## 核心内容详解

### 提示词结构

```typescript
export function buildConsolidationPrompt(
  memoryRoot: string,      // 记忆根目录
  transcriptDir: string,   // 会话记录目录
  extra: string,           // 额外上下文
): string
```

### 四阶段整合流程

#### Phase 1 — Orient（定向）
- `ls` 记忆目录查看已有内容
- 阅读 `CLAUDE.md` 了解当前索引
- 浏览现有主题文件避免重复创建
- 检查 `logs/` 或 `sessions/` 子目录

#### Phase 2 — Gather recent signal（收集信号）

**优先级排序的信息源**:
1. **每日日志** (`logs/YYYY/MM/YYYY-MM-DD.md`) — 追加式流
2. **漂移的现有记忆** — 与代码库当前状态矛盾的事实
3. **会话记录搜索** — 使用 grep 搜索特定上下文

**搜索示例**:
```bash
grep -rn "<narrow term>" ${transcriptDir}/ --include="*.jsonl" | tail -50
```

#### Phase 3 — Consolidate（整合）

**操作**:
- 将新信号合并到现有主题文件而非创建近似重复
- 将相对日期（"昨天"、"上周"）转换为绝对日期
- 删除已被证伪的事实
- 遵循系统提示词的自动记忆格式约定

#### Phase 4 — Prune and index（修剪和索引）

**更新 `CLAUDE.md`**:
- 保持在 ${MAX_ENTRYPOINT_LINES} 行以内
- 保持在 ~25KB 以内
- 每项一行，约150字符：`- [Title](file.md) — one-line hook`
- 从不直接写入记忆内容

**操作**:
- 移除指向过时、错误或已被取代记忆的指针
- 降级冗长条目（>200字符）
- 添加指向新重要记忆的指针
- 解决矛盾

### 依赖的常量

从 `memdir/memdir.js` 导入：
- `DIR_EXISTS_GUIDANCE`: 目录存在指导
- `ENTRYPOINT_NAME`: 入口文件名（`CLAUDE.md`）
- `MAX_ENTRYPOINT_LINES`: 最大入口行数

### 额外上下文

`extra` 参数用于注入工具约束和会话列表：

```
**Tool constraints for this run:** Bash is restricted to read-only commands...

Sessions since last consolidation (N):
- session-id-1
- session-id-2
...
```

## 设计要点

1. **独立性**: 从 dream.ts 提取，避免依赖 KAIROS 特性标志
2. **结构化流程**: 四阶段清晰的工作流程
3. **增量更新**: 优先合并而非创建新文件
4. **日期规范化**: 相对日期转绝对日期
5. **索引维护**: 保持入口文件精简

## 与其他文件的关系

- **memdir/memdir.js**: 提供目录指导常量和入口文件名

## 注意事项

1. **只读限制**: 自动梦境中 Bash 工具限制为只读命令
2. **会话提示**: extra 中包含会话列表供参考
3. **手动/自动差异**: 手动 `/dream` 在主循环运行，有正常权限
