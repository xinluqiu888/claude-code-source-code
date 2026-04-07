# index.ts — 反馈命令配置

> **一句话总结**：定义 `/feedback` 和 `/bug` 命令的元数据配置，支持条件启用。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/feedback/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 26 行 |
| 主要职责 | 导出feedback命令的配置对象，包含复杂的启用条件 |

---

## 功能概述

该文件是反馈命令的入口配置。与简单命令不同，该配置包含复杂的 `isEnabled` 逻辑，根据多种环境变量和状态决定命令是否可用。

---

## 核心内容详解

### 导入与依赖

```typescript
import type { Command } from '../../commands.js'
import { isPolicyAllowed } from '../../services/policyLimits/index.js'
import { isEnvTruthy } from '../../utils/envUtils.js'
import { isEssentialTrafficOnly } from '../../utils/privacyLevel.js'
```

### 命令配置对象

- **类型**: `Command`
- **配置项**:
  - `aliases`: `['bug']` - 命令别名
  - `type`: `'local-jsx'` - 本地JSX命令类型
  - `name`: `'feedback'` - 命令名称
  - `description`: `'Submit feedback about Claude Code'` - 命令描述
  - `argumentHint`: `'[report]'` - 参数提示（可选报告内容）
  - `isEnabled`: 复杂函数，检查多个禁用条件
  - `load`: `() => import('./feedback.js')` - 懒加载执行函数

### isEnabled 逻辑详解

命令在以下任一条件为真时被**禁用**：

1. `CLAUDE_CODE_USE_BEDROCK` - 使用AWS Bedrock时
2. `CLAUDE_CODE_USE_VERTEX` - 使用Google Vertex时
3. `CLAUDE_CODE_USE_FOUNDRY` - 使用Foundry时
4. `DISABLE_FEEDBACK_COMMAND` - 显式禁用反馈命令
5. `DISABLE_BUG_COMMAND` - 显式禁用bug命令
6. `isEssentialTrafficOnly()` - 仅允许必要流量模式
7. `USER_TYPE === 'ant'` - 特定用户类型
8. `!isPolicyAllowed('allow_product_feedback')` - 策略不允许反馈

### 对外导出

- **默认导出**: `feedback` 命令配置对象

---

## 设计要点

1. **多条件禁用**: 复杂的 `isEnabled` 逻辑确保在特定环境下禁用反馈功能。

2. **策略控制**: 使用 `isPolicyAllowed` 允许通过策略系统动态控制功能可用性。

3. **隐私保护**: `isEssentialTrafficOnly` 检查确保在隐私模式下不发送反馈。

4. **双别名**: 同时支持 `/feedback` 和 `/bug` 两个命令，后者更直观用于报告问题。

---

## 与其他文件的关系

**依赖**:
- `Command` 类型 (`../../commands.js`)
- `isPolicyAllowed` (`../../services/policyLimits/index.js`)
- `isEnvTruthy` (`../../utils/envUtils.js`)
- `isEssentialTrafficOnly` (`../../utils/privacyLevel.js`)

**被依赖**:
- 命令注册系统

---

## 注意事项

1. **环境变量敏感**: 多个环境变量可以禁用此功能，需要在部署时仔细检查。

2. **策略检查**: `allow_product_feedback` 策略检查表明反馈功能可以被组织策略控制。

3. **隐私模式**: `isEssentialTrafficOnly` 确保在隐私敏感模式下不会发送用户反馈到服务器。
