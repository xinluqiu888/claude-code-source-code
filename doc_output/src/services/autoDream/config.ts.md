# config.ts — 自动梦境配置

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/autoDream/config.ts`
- **作用域**: 自动梦境功能开关配置
- **主要导出**:
  - `isAutoDreamEnabled`: 检查自动梦境是否启用

## 功能概述

判断后台记忆整合功能是否应运行。用户设置（`settings.json` 中的 `autoDreamEnabled`）优先于 GrowthBook 默认值；未明确设置时回退到 `tengu_onyx_plover` 特性开关。

## 核心内容详解

### 配置优先级

```
用户设置 (autoDreamEnabled) → GrowthBook (tengu_onyx_plover.enabled)
```

1. **用户设置优先**: 如果 `settings.json` 中明确定义了 `autoDreamEnabled`，直接使用该值
2. **特性开关回退**: 未设置时使用 GrowthBook 的 `tengu_onyx_plover` 特性值

### 核心函数

#### `isAutoDreamEnabled()`
检查自动梦境是否启用：

**返回值**: `boolean`

**逻辑**:
```typescript
export function isAutoDreamEnabled(): boolean {
  const setting = getInitialSettings().autoDreamEnabled
  if (setting !== undefined) return setting
  const gb = getFeatureValue_CACHED_MAY_BE_STALE<{ enabled?: unknown } | null>(
    'tengu_onyx_plover',
    null,
  )
  return gb?.enabled === true
}
```

## 设计要点

1. **叶子模块**: 故意最小化导入，使 UI 组件可以读取状态而不拖入 `autoDream.ts` 的依赖链
2. **配置优先**: 用户显式配置覆盖服务端特性开关
3. **类型安全**: GrowthBook 返回值进行未知类型检查

## 与其他文件的关系

- **utils/settings/settings.ts**: 提供 `getInitialSettings`
- **services/analytics/growthbook.ts**: 提供 `getFeatureValue_CACHED_MAY_BE_STALE`

## 注意事项

1. **缓存值**: 使用 `CACHED_MAY_BE_STALE` 后缀的函数，值可能不是最新的
2. **严格相等**: 检查 `gb?.enabled === true`，避免 truthy 值误判
