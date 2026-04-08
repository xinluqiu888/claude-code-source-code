# shellHistoryCompletion.ts — Shell 历史命令补全

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/suggestions/shellHistoryCompletion.ts`  
**关联模块**: 历史记录管理、调试工具  
**主要依赖**: `src/history.js`, `src/utils/debug.js`

## 功能概述

本文件实现基于 shell 历史记录的命令补全功能：
- 从历史记录中提取以 `!` 开头的 bash 命令
- 提供智能前缀匹配补全建议
- 缓存机制减少重复读取
- 支持向缓存前置新命令

## 核心内容详解

### 类型定义

```typescript
type ShellHistoryMatch = {
  fullCommand: string  // 历史记录中的完整命令
  suffix: string       // 匹配后剩余部分（幽灵文本显示）
}
```

### 核心函数

#### `getShellHistoryCommands()`
获取 shell 历史命令列表：
- 使用60秒TTL的内存缓存
- 从历史记录中筛选以 `!` 开头的条目
- 去除 `!` 前缀，去重，最多保留50条
- 缓存 `shellHistoryCache` 和 `shellHistoryCacheTimestamp`

#### `getShellHistoryCompletion(input)`
获取历史命令补全建议：
- 输入长度小于2或仅空白字符时返回 null
- 查找以输入内容开头的第一个命令
- 要求精确前缀匹配（包含空格）
- 返回匹配项的 `fullCommand` 和 `suffix`

#### `clearShellHistoryCache()`
清除历史命令缓存，用于历史记录更新后刷新。

#### `prependToShellHistoryCache(command)`
向缓存前置新命令：
- 如果命令已存在则移动到最前面（去重）
- 缓存未初始化时为无操作（下次读取会获取完整历史）

## 设计要点

1. **缓存策略**:
   - TTL: 60秒，合理平衡实时性和性能
   - 基于历史记录在输入过程中不会改变的假设

2. **前缀匹配规则**:
   - 使用 `startsWith(input)` 精确匹配
   - 确保 `"ls "` 匹配 `"ls -lah"` 但 `"ls  "`（两个空格）不匹配
   - 排除与输入完全相同的命令

3. **历史记录来源**:
   - 从 `getHistory()` 获取所有历史条目
   - 筛选 `entry.display.startsWith('!')` 的条目
   - 仅支持 bash 风格的 `!command` 历史扩展

## 与其他文件的关系

- **history.ts**: 提供历史记录遍历接口
- **debug.ts**: 用于记录调试日志

## 注意事项

- 空输入或短输入（<2字符）不会触发补全
- 匹配区分大小写（基于原始命令）
- 仅返回第一个匹配项（最早的历史记录）
- 缓存未初始化时 `prependToShellHistoryCache` 不执行任何操作
- 调试日志使用 `logForDebugging`，仅在调试模式下输出
