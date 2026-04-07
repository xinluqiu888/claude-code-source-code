# effort.tsx — 模型使用努力级别设置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/effort/effort.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 主要职责 | 提供模型使用努力级别设置界面 |

## 功能概述

该文件实现了 `/effort` 命令，用于设置模型使用的努力级别。支持 `low`、`medium`、`high`、`max`、`auto` 五个级别，影响模型在任务上的推理深度和全面性。

## 核心内容详解

### 努力级别

| 级别 | 描述 |
|------|------|
| `low` | 快速、直接的实现 |
| `medium` | 平衡方法，标准测试 |
| `high` | 全面的实现，广泛测试 |
| `max` | 最大能力，最深推理（仅限 Opus 4.6） |
| `auto` | 使用模型的默认努力级别 |

### 主要函数

**setEffortValue**
- 设置努力级别
- 持久化到用户设置
- 记录分析事件
- 处理环境变量覆盖

**showCurrentEffort**
- 显示当前努力级别
- 考虑环境变量覆盖
- 显示当前生效级别

**unsetEffortLevel**
- 重置为 auto
- 清除用户设置
- 处理环境变量冲突

**executeEffort**
- 执行命令逻辑
- 处理 `auto` 和 `unset`
- 验证级别有效性

### 组件

**ShowCurrentEffort**
- 使用 `useAppState` 获取当前值
- 使用 `useMainLoopModel` 获取当前模型
- 显示当前努力级别

**ApplyEffortAndClose**
- 使用 `useSetAppState` 更新状态
- 使用 `useEffect` 应用更改
- 调用 onDone 完成

## 设计要点

1. **环境变量覆盖**：`CLAUDE_CODE_EFFORT_LEVEL` 可覆盖设置
2. **持久化**：设置保存到用户设置
3. **模型感知**：显示的努力级别考虑当前模型
4. **即时反馈**：更改后立即生效

## 与其他文件的关系

- **effort.ts**: 提供努力级别工具函数
- **Settings/**: 设置持久化
- **index.ts**: 命令注册
