# passes/index.ts

## 文件描述
Guest Passes 功能配置 - 根据推荐奖励状态动态显示描述信息

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | passes |
| 描述 | 动态描述（基于referrerReward状态） |
| 可用性 | claude-ai |
| 功能 | 配置Guest Passes命令，管理访客通行证功能 |

## 函数概述

### getDescription
动态生成命令描述：
- 如果有推荐奖励：`Share guest passes and get $50 per signup`
- 否则：`Share guest passes`

### isEnabled
根据 `DISABLE_PASSES_COMMAND` 环境变量和 `getReferrerReward()` 返回值决定是否启用该命令

## 核心内容

### 命令配置
- 类型：local-jsx
- 名称：passes
- 描述：动态生成，基于推荐奖励状态
- 可用性：claude-ai
- 加载方式：动态导入 `./passes.js`

### 条件渲染
```typescript
isEnabled: () =>
  !isEnvTruthy(process.env.DISABLE_PASSES_COMMAND) && getReferrerReward() !== null,
```

## 设计点

1. **动态描述**：根据用户是否有推荐奖励动态显示不同的命令描述
2. **条件启用**：通过环境变量和功能状态控制命令是否可用
3. **懒加载**：使用动态导入延迟加载组件代码

## 与其他文件的关系

- 导入 `./passes.js` 获取实际组件实现
- 依赖 `../../utils/auth.js` 中的 `getReferrerReward` 函数
- 依赖 `../../utils/envUtils.js` 中的 `isEnvTruthy` 函数

## 注意事项

- 仅在 `claude-ai` 环境中可用
- 需要用户有推荐奖励才能显示完整描述
- 可通过 `DISABLE_PASSES_COMMAND` 环境变量禁用
