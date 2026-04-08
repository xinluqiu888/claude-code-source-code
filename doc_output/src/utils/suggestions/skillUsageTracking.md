# skillUsageTracking.ts — 技能使用频率跟踪

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/suggestions/skillUsageTracking.ts`  
**关联模块**: 全局配置、命令建议  
**主要依赖**: `src/utils/config.js`

## 功能概述

本文件实现技能使用频率跟踪机制，用于优化命令建议排序：
- 记录技能使用次数和时间戳
- 计算基于使用频率和时效性的综合分数
- 使用指数衰减模型（7天半衰期）
- 防抖机制避免频繁写入配置文件

## 核心内容详解

### 常量定义

```typescript
const SKILL_USAGE_DEBOUNCE_MS = 60_000  // 1分钟防抖间隔
```

### 数据结构

技能使用数据存储在全局配置中：
```typescript
type SkillUsageData = {
  [skillName: string]: {
    usageCount: number   // 累计使用次数
    lastUsedAt: number   // 最后使用时间戳（毫秒）
  }
}
```

### 核心函数

#### `recordSkillUsage(skillName)`
记录技能使用情况：
- 使用进程级 Map `lastWriteBySkill` 进行防抖
- 同一技能60秒内多次调用只记录一次
- 更新全局配置中的 `skillUsage` 字段
- 每次调用增加 `usageCount` 并更新 `lastUsedAt`

#### `getSkillUsageScore(skillName): number`
计算技能使用分数：
- 基于使用次数和时效性
- 使用指数衰减公式：`usageCount * recencyFactor`
- 半衰期7天：7天前的使用价值减半
- 最小时效系数0.1，避免旧技能完全沉没

分数计算公式：
```
daysSinceUse = (now - lastUsedAt) / (1000 * 60 * 60 * 24)
recencyFactor = max(0.5^(daysSinceUse / 7), 0.1)
score = usageCount * recencyFactor
```

## 设计要点

1. **防抖机制**: 
   - 进程级 Map 缓存最后写入时间
   - 60秒防抖间隔避免频繁文件I/O
   - 参考 `config.ts` 中的 `lastConfigStatTime` 模式

2. **指数衰减模型**:
   - 7天半衰期平衡近期使用和历史积累
   - 最小值0.1确保偶尔使用的老技能仍有展示机会

3. **持久化存储**: 
   - 使用 `saveGlobalConfig` 存储到用户配置
   - 数据跨会话持久化

## 与其他文件的关系

- **config.ts**: 提供全局配置的读写接口
- **commandSuggestions.ts**: 调用 `getSkillUsageScore` 用于命令排序

## 注意事项

- 防抖基于进程生命周期，重启后首次使用会立即记录
- 分数计算使用浮点数，排序时可能出现精度问题
- 全局配置中的 `skillUsage` 对象可能随时间增长，建议定期清理
- 仅 `prompt` 类型命令（技能）会被跟踪，内置命令不计入
