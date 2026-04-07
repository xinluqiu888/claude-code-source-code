# extra-usage-noninteractive.ts — 额外使用量配置（非交互式）

> **一句话总结**：非交互式会话处理额外使用量配置，返回文本结果。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/extra-usage/extra-usage-noninteractive.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 16 行 |
| 主要职责 | 为非交互式会话提供额外使用量配置功能 |

---

## 功能概述

该文件是 `/extra-usage` 命令的非交互式版本。在非交互式会话（如CI/CD管道）中，不使用React组件，而是直接返回文本结果。适用于脚本自动化场景。

---

## 核心内容详解

### 导入与依赖

```typescript
import { runExtraUsage } from './extra-usage-core.js'
```

### 主要函数

#### call(): Promise<{ type: 'text'; value: string }>

- **类型**: `async function`
- **返回值**: `Promise<{ type: 'text'; value: string }>`
- **用途**: 执行额外使用量配置并返回文本结果

**执行流程**:

1. 调用 `runExtraUsage()` 执行核心逻辑
2. 如果返回类型为 `'message'`，直接返回文本结果
3. 如果返回类型为 `'browser-opened'`：
   - 如果浏览器成功打开，返回告知用户已打开浏览器的消息
   - 如果浏览器打开失败，返回包含URL的文本消息，提示用户手动访问

---

## 设计要点

1. **纯文本输出**: 非交互式版本不使用任何UI组件，只返回文本。

2. **统一核心逻辑**: 复用 `runExtraUsage` 的核心逻辑，与交互式版本保持一致。

3. **浏览器结果处理**: 即使无法显示UI，也会告知用户浏览器操作结果。

---

## 与其他文件的关系

**依赖**:
- `runExtraUsage` (`./extra-usage-core.js`) - 核心逻辑

**被依赖**:
- `extra-usage/index.ts` - 注册为非交互式命令

---

## 注意事项

1. **浏览器状态反馈**: 在非交互式环境中，浏览器是否真正打开无法直观确认，因此返回明确的文本说明。

2. **降级提示**: 如果浏览器未打开，提供可手动访问的URL。
