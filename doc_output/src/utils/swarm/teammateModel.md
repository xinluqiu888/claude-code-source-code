# teammateModel.ts — 队友模型配置

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/teammateModel.ts`  
**关联模块**: 模型配置、API 提供商  
**主要依赖**: `src/utils/model/configs.js`, `src/utils/model/providers.js`

## 功能概述

本文件提供 teammates 的默认模型回退配置。当用户从未在 `/config` 中设置 `teammateDefaultModel` 时，新 teammates 使用此回退模型。

## 核心内容详解

### 核心函数

#### `getHardcodedTeammateModelFallback(): string`
获取硬编码的 teammate 模型回退值：
- 使用 `CLAUDE_OPUS_4_6_CONFIG` 根据当前 API 提供商获取正确的模型 ID
- 支持 Bedrock、Vertex、Foundry 等提供商

### 实现细节

```typescript
export function getHardcodedTeammateModelFallback(): string {
  return CLAUDE_OPUS_4_6_CONFIG[getAPIProvider()]
}
```

`CLAUDE_OPUS_4_6_CONFIG` 是一个记录，键为提供商名称，值为该提供商对应的模型 ID。

## 设计要点

1. **提供商感知**: 不同提供商（Bedrock、Vertex、Foundry）有不同的模型 ID，必须返回正确的 ID
2. **硬编码回退**: 当用户配置缺失时提供可靠的默认值
3. **模型发布更新**: 注释标记 `@MODEL_LAUNCH` 提醒在新模型发布时更新此回退

## 与其他文件的关系

- **model/configs.ts**: 提供 `CLAUDE_OPUS_4_6_CONFIG`
- **model/providers.ts**: 提供 `getAPIProvider()`
- **spawnInProcess.ts**: 在启动 teammate 时可能使用模型配置

## 注意事项

- 此回退仅在 `teammateDefaultModel` 未设置时生效
- 用户可以通过设置文件覆盖此默认值
- 新模型发布时必须更新此文件
