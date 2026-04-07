# index.ts — Mobile命令配置

> **一句话总结**：定义 `/mobile` 命令的元数据配置，支持ios和android别名。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/mobile/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 12 行 |
| 主要职责 | 导出mobile命令的配置对象 |

---

## 功能概述

该文件是Mobile命令的入口配置。用户可以通过 `/mobile`、`/ios` 或 `/android` 命令显示二维码，用于下载Claude移动应用到iOS或Android设备。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'mobile'` - 命令名称
  - `aliases`: `['ios', 'android']` - 命令别名
  - `description`: `'Show QR code to download the Claude mobile app'` - 命令描述
  - `load`: `() => import('./mobile.js')` - 懒加载执行函数

### 对外导出

- **默认导出**: `mobile` 命令配置对象

---

## 设计要点

1. **多别名支持**: 支持 `mobile`、`ios`、`android` 三个命令名称。

2. **QR码展示**: 使用JSX类型展示QR码，方便用户扫码下载。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **平台切换**: 界面支持在iOS和Android之间切换显示不同的QR码。

2. **别名等效**: `/ios` 和 `/android` 都会打开相同的界面，只是默认选中的平台不同。
