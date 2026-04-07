# config.ts — 分析服务配置管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/config.ts`
- **所属模块**: Analytics Service
- **功能类型**: 配置管理

## 功能概述

该文件提供分析服务（Analytics）的全局配置管理功能，主要用于确定何时应禁用分析收集。它是所有分析系统（Datadog、第一方事件日志）的共享配置源。

## 核心内容详解

### 主要函数

#### `isAnalyticsDisabled(): boolean`
检查分析操作是否应该被禁用。在以下情况下禁用分析：
- 测试环境（`NODE_ENV === 'test'`）
- 使用第三方云服务提供商（Bedrock/Vertex/Foundry）
- 隐私级别设置为"无遥测"或"仅必要流量"

#### `isFeedbackSurveyDisabled(): boolean`
检查是否应禁用反馈调查。与 `isAnalyticsDisabled()` 不同，此方法不会在第三方提供商上阻止。

### 依赖关系

- `../../utils/envUtils.ts` — 环境变量工具
- `../../utils/privacyLevel.ts` — 隐私级别管理

## 设计要点

1. **集中式配置** — 单一数据源决定所有分析系统的启用状态
2. **安全优先** — 默认在测试和第三方云环境中禁用，保护用户隐私
3. **模块化设计** — 与具体的分析实现分离，避免循环依赖

## 与其他文件的关系

- **被引用**: `datadog.ts`, `firstPartyEventLogger.ts`, `sink.ts` 等分析模块
- **依赖**: `envUtils.ts`, `privacyLevel.ts`

## 注意事项

- 该文件设计为无依赖（除了工具函数），避免导入循环
- 第三方云提供商的检测基于环境变量 `CLAUDE_CODE_USE_*`
- 隐私级别检查独立于云提供商检测，确保双重保护
