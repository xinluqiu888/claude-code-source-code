# prompt.ts — ScheduleCron工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/ScheduleCronTool/prompt.ts`
- **作用**: 定义Cron调度系统的功能开关和提示构建函数

## 功能概述

该文件提供Cron调度系统的统一控制入口，包括功能开关检测、工具名称常量和动态提示构建。

## 核心内容详解

### 功能开关检测

#### isKairosCronEnabled

```typescript
export function isKairosCronEnabled(): boolean
```

统一控制Cron调度系统。结合构建时`feature('AGENT_TRIGGERS')`标志和运行时`tengu_kairos_cron` GrowthBook开关。

检测逻辑：
1. 首先检查`AGENT_TRIGGERS`构建时标志
2. 检查`CLAUDE_CODE_DISABLE_CRON`环境变量覆盖
3. 查询GrowthBook的`tengu_kairos_cron`开关（5分钟刷新窗口）
4. 默认值为`true`（/loop功能已GA）

#### isDurableCronEnabled

```typescript
export function isDurableCronEnabled(): boolean
```

控制持久化Cron任务的开关。比`isKairosCronEnabled`更窄，仅控制`durable: true`的任务。

### 工具名称常量

```typescript
export const CRON_CREATE_TOOL_NAME = 'CronCreate'
export const CRON_DELETE_TOOL_NAME = 'CronDelete'
export const CRON_LIST_TOOL_NAME = 'CronList'
```

### 提示构建函数

#### buildCronCreateDescription/Prompt

根据`durableEnabled`参数动态构建描述和提示：
- `durableEnabled: true`: 说明支持持久化到`.claude/scheduled_tasks.json`
- `durableEnabled: false`: 仅支持会话内调度

### Cron提示内容要点

1. **标准5字段Cron**: `M H DoM Mon DoW`（分钟 小时 日 月 星期）
2. **一次性任务**: `recurring: false`，触发后自动删除
3. **周期性任务**: `recurring: true`（默认），7天后自动过期
4. **避免整点**: 建议使用非0/30分钟值，避免集群负载峰值
5. **持久化选项**: `durable: true`写入磁盘，会话重启后恢复

## 设计要点

1. **分层控制**: `isKairosCronEnabled`控制整体，`isDurableCronEnabled`控制持久化
2. **动态内容**: 根据特性开关动态调整提示文本
3. **本地覆盖**: `CLAUDE_CODE_DISABLE_CRON`可强制关闭
4. **默认开启**: /loop功能已GA，默认启用

## 与其他文件的关系

- **CronCreateTool.ts/CronDeleteTool.ts/CronListTool.ts**: 导入使用这些函数
- **growthbook.ts**: GrowthBook特性查询
- **cronTasks.ts**: `DEFAULT_CRON_JITTER_CONFIG`

## 注意事项

- 周期性任务默认7天后自动过期
- 调度器只在REPL空闲时触发任务
- 调度器会添加少量确定性抖动（最大15分钟）
