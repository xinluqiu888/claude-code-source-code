# thinkback-play/index.ts

## 文件描述
Thinkback Play 命令配置 - 播放回顾动画

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | thinkback-play |
| 描述 | Play the thinkback animation |
| 隐藏 | true |
| 启用条件 | tengu_thinkback feature |
| 支持非交互 | false |

## 核心内容

### 命令配置
```typescript
const thinkbackPlay = {
  type: 'local',
  name: 'thinkback-play',
  description: 'Play the thinkback animation',
  isEnabled: () =>
    checkStatsigFeatureGate_CACHED_MAY_BE_STALE('tengu_thinkback'),
  isHidden: true,
  supportsNonInteractive: false,
  load: () => import('./thinkback-play.js'),
} satisfies Command
```

### 说明
- 隐藏命令，仅播放动画
- 由 thinkback skill 在生成完成后调用

## 设计点

1. **隐藏命令**：不显示在帮助中
2. **技能调用**：由 thinkback skill 触发
3. **功能标志控制**：通过 Statsig 控制
4. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `checkStatsigFeatureGate_CACHED_MAY_BE_STALE` 从 `../../services/analytics/growthbook.js`

## 注意事项

- 内部使用命令
- 播放年度回顾动画
- 由 skill 自动调用
