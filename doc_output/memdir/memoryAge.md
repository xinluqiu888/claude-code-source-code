# memoryAge.ts — 内存年龄计算

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/memoryAge.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

计算内存文件年龄的工具函数，提供天数计算、人类可读格式和新鲜度警告。帮助模型理解内存的时效性。

## 核心内容详解

### 主要导出

1. **memoryAgeDays(mtimeMs)** — 计算内存年龄天数
   - 参数: mtimeMs (修改时间毫秒戳)
   - 返回: 天数 (向下取整)
   - 0 = 今天, 1 = 昨天, 2+ = 更旧
   - 负输入 (未来时间、时钟偏移) 限制为 0

2. **memoryAge(mtimeMs)** — 人类可读年龄字符串
   - 参数: mtimeMs (修改时间毫秒戳)
   - 返回: "today", "yesterday", 或 "{d} days ago"
   - 模型擅长理解这种格式而非原始 ISO 时间戳

3. **memoryFreshnessText(mtimeMs)** — 纯文本新鲜度警告
   - 参数: mtimeMs (修改时间毫秒戳)
   - 返回: 警告文本或空字符串
   - 仅对 >1 天的内存返回警告
   - 用于消费者提供自己包装的场景

4. **memoryFreshnessNote(mtimeMs)** — 带包装的的新鲜度注释
   - 参数: mtimeMs (修改时间毫秒戳)
   - 返回: `<system-reminder>` 包装的警告或空字符串
   - 用于调用者不提供自己包装的场景 (如 FileReadTool 输出)

## 年龄阈值

| 天数 | memoryAgeDays | memoryAge | 新鲜度警告 |
|------|--------------|-----------|-----------|
| 0    | 0            | today     | 无        |
| 1    | 1            | yesterday | 无        |
| 2+   | 2+           | N days ago| 有        |

## 设计要点

1. **向下取整**:
   - 使用 Math.floor 而非 Math.round
   - 0 表示今天 (任何时间)
   - 1 表示昨天

2. **负值处理**:
   - Math.max(0, ...) 限制为 0
   - 处理未来时间戳和时钟偏移

3. **新鲜度触发**:
   - 仅对 >1 天 (2+天) 的内存显示警告
   - 今天和昨天的内存视为新鲜

4. **两种格式**:
   - `memoryFreshnessText`: 纯文本，消费者自行包装
   - `memoryFreshnessNote`: 带 `<system-reminder>` 包装

5. **警告内容**:
   - 说明内存是时间点观察
   - 提示代码行为或文件行号引用可能过时
   - 建议验证当前代码

## 与其他文件的关系

- **messages.ts**: 使用 relevant_memories → wrapMessagesInSystemReminder
- **FileReadTool**: 使用 memoryFreshnessNote

## 注意事项

1. 模型不擅长日期算术，人类可读格式更易理解
2. 引用格式 (file:line) 的过时代码声明特别危险
3. 新鲜度警告旨在减少过时代码状态记忆的错误断言
4. 仅对 >1 天的内存显示警告避免过度噪声
5. 时钟偏移不会导致负年龄
