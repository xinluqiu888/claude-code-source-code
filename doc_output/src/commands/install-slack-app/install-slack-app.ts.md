# install-slack-app.ts — 安装Claude Slack应用

> **一句话总结**：打开浏览器引导用户安装Claude Slack应用到他们的Slack工作区。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/install-slack-app/install-slack-app.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 30 行 |
| 主要职责 | 打开浏览器到Slack应用市场页面 |

---

## 功能概述

该文件实现了 `/install-slack-app` 命令，引导用户安装Claude Slack应用。命令打开浏览器到Slack应用市场页面，并跟踪安装尝试次数。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { LocalCommandResult } from '../../commands.js'
import { logEvent } from '../../services/analytics/index.js'
import { openBrowser } from '../../utils/browser.js'
import { saveGlobalConfig } from '../../utils/config.js'
```

### 常量

- **SLACK_APP_URL**: `'https://slack.com/marketplace/A08SF47R6P4-claude'` - Claude Slack应用市场URL

### 主要函数

#### call(): Promise<LocalCommandResult>

- **类型**: `async function`
- **返回值**: `Promise<LocalCommandResult>`
- **用途**: 主入口函数

**执行流程**:

1. 记录分析事件：`logEvent('tengu_install_slack_app_clicked', {})`
2. 更新全局配置：增加 `slackAppInstallCount` 计数器
3. 打开浏览器：`openBrowser(SLACK_APP_URL)`
4. 返回成功或失败的文本结果

---

## 设计要点

1. **分析追踪**: 记录安装尝试次数用于分析用户行为。

2. **浏览器打开**: 使用统一工具函数打开浏览器，确保跨平台兼容。

3. **计数器**: 跟踪安装尝试次数，可能用于提示或限制。

---

## 与其他文件的关系

**依赖**:
- `logEvent` (`../../services/analytics/index.js`)
- `openBrowser` (`../../utils/browser.js`)
- `saveGlobalConfig` (`../../utils/config.js`)

**被依赖**:
- `install-slack-app/index.ts` - 导出为命令配置

---

## 注意事项

1. **Slack应用市场**: 需要用户有Slack账户和权限才能安装应用。

2. **浏览器依赖**: 依赖系统能够打开浏览器，某些环境中可能失败。

3. **计数器递增**: 每次点击都递增计数器，包括重复尝试。
