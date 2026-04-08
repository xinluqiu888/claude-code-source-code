# commandSuggestions.ts — 斜杠命令自动补全与搜索

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/suggestions/commandSuggestions.ts`  
**关联模块**: 命令系统、提示输入组件、技能使用跟踪  
**主要依赖**: `fuse.js` (模糊搜索), `src/commands.ts`, `src/components/PromptInput/PromptInputFooterSuggestions.js`

## 功能概述

本文件实现了一个完整的斜杠命令（slash command）智能提示系统，支持：
- 基于模糊搜索的命令查找（使用 Fuse.js）
- 命令别名匹配和优先级排序
- 最近使用技能的排序展示
- 输入中斜杠命令的位置检测与高亮
- 智能补全建议生成

## 核心内容详解

### 主要类型定义

```typescript
type CommandSearchItem = {
  descriptionKey: string[]      // 描述关键词数组
  partKey: string[] | undefined // 命令分段关键词
  commandName: string           // 命令名称
  command: Command              // 命令对象
  aliasKey: string[] | undefined // 别名数组
}

type MidInputSlashCommand = {
  token: string           // 完整命令token，如 "/com"
  startPos: number        // "/" 在输入中的位置
  partialCommand: string  // 部分命令，如 "com"
}
```

### 核心函数

#### `generateCommandSuggestions(input, commands)`
生成命令建议列表的核心函数：
- 空输入时按类别分组展示（最近使用、内置、用户、项目、策略、其他）
- 有输入时使用 Fuse.js 进行模糊搜索
- 支持多维度排序：精确匹配 > 前缀匹配 > 模糊匹配 > 使用频率

#### `getBestCommandMatch(partialCommand, commands)`
查找最佳命令匹配，用于内联补全幽灵文本：
- 仅返回前缀匹配的建议
- 返回匹配的后缀和完整命令名

#### `findMidInputSlashCommand(input, cursorOffset)`
检测输入中光标位置是否处于斜杠命令中（非输入开头）：
- 使用正则 `/\s\/([a-zA-Z0-9_:-]*)$/` 匹配
- 返回命令位置和token信息

#### `applyCommandSuggestion(suggestion, shouldExecute, ...)`
应用选中的命令建议到输入框：
- 更新输入内容和光标位置
- 如命令无需参数则自动执行

#### `findSlashCommandPositions(text)`
查找文本中所有斜杠命令的位置，用于语法高亮。

## 设计要点

1. **Fuse.js 索引缓存**: 使用 `fuseCache` 缓存搜索索引，以命令数组引用作为键，避免每次输入都重建索引

2. **多级排序策略**: 搜索结果按以下优先级排序：
   - 精确名称匹配（最高）
   - 精确别名匹配
   - 前缀名称匹配
   - 前缀别名匹配
   - Fuse 模糊匹配分数
   - 技能使用频率（用于打破平局）

3. **隐藏命令处理**: 当用户输入隐藏的命令名称时，即使该命令已被隐藏，也会将其前置到搜索结果中（如果无同名可见命令）

4. **命令ID唯一性**: 对于可能重复的 prompt 类型命令（来自不同源），使用 `name:source:repository` 格式生成唯一ID

## 与其他文件的关系

- **commands.ts**: 导入命令类型定义和相关工具函数
- **PromptInputFooterSuggestions.js**: 导出 `SuggestionItem` 类型
- **skillUsageTracking.ts**: 获取技能使用频率分数用于排序

## 注意事项

- 搜索阈值设为 0.3（较严格），优先准确匹配
- 别名仅在用户输入匹配时显示在括号中
- 支持命令名中的分隔符（`:`、`-`、`_`），这些字符被视为单词边界
- 使用 YARR JIT 友好的正则表达式（避免 lookbehind）
