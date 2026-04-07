# thinkback/index.ts

## 文件描述
Thinkback 命令配置 - 年度回顾

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | think-back |
| 描述 | Your 2025 Claude Code Year in Review |
| 启用条件 | tengu_thinkback feature |

## 核心内容

### 命令配置
```typescript
const thinkback = {
  type: 'local-jsx',
  name: 'think-back',
  description: 'Your 2025 Claude Code Year in Review',
  isEnabled: () =>
    checkStatsigFeatureGate_CACHED_MAY_BE_STALE('tengu_thinkback'),
  load: () => import('./thinkback.js'),
} satisfies Command
```

## 设计点

1. **功能标志控制**：通过 Statsig 控制
2. **年度回顾**：显示用户使用统计
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `checkStatsigFeatureGate_CACHED_MAY_BE_STALE` 从 `../../services/analytics/growthbook.js`

## 注意事项

- 2025年度回顾功能
- 需要功能标志启用
- 显示使用统计和亮点
