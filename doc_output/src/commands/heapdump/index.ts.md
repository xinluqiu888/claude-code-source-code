# index.ts — 堆内存导出命令配置

> **一句话总结**：定义 `/heapdump` 命令的元数据配置，这是一个隐藏的调试命令。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/heapdump/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 12 行 |
| 主要职责 | 导出heapdump命令的配置对象 |

---

## 功能概述

该文件是堆内存导出命令的入口配置。这是一个隐藏命令，用于内存调试，普通用户不应看到此命令。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local'` - 本地命令类型
  - `name`: `'heapdump'` - 命令名称
  - `description`: `'Dump the JS heap to ~/Desktop'` - 命令描述
  - `isHidden`: `true` - 在帮助中隐藏
  - `supportsNonInteractive`: `true` - 支持非交互式会话
  - `load`: `() => import('./heapdump.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `heapDump` 命令配置对象

---

## 设计要点

1. **隐藏命令**: `isHidden: true` 确保普通用户看不到此命令。

2. **非交互式支持**: 支持在CI/CD等自动化环境中使用。

3. **简单直接**: 无参数，直接执行导出操作。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **调试专用**: 仅供开发团队内存分析使用。

2. **桌面路径**: 输出路径固定为 `~/Desktop`，确保可预测。

3. **懒加载**: 堆导出服务可能包含大量依赖，使用懒加载减少启动时间。
