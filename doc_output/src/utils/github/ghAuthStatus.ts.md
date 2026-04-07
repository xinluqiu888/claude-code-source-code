# ghAuthStatus.ts — GitHub CLI认证状态检测

> **一句话总结**：检测gh CLI的安装和认证状态，用于遥测数据收集。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/github/ghAuthStatus.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~30行 |
| 主要职责 | 检测gh CLI状态和认证 |

---

## 功能概述

该模块提供轻量级的gh CLI状态检测：
1. 使用`which()`检测gh是否安装
2. 使用`gh auth token`检测是否已认证

使用`auth token`而非`auth status`是因为后者会发起网络请求，而前者只读取本地配置。

---

## 核心内容详解

### 导出类型

- **`GhAuthStatus`**
  - 联合类型：`'authenticated' | 'not_authenticated' | 'not_installed'`

### 导出函数

- **`getGhAuthStatus`**
  - 返回值：`Promise<GhAuthStatus>`
  - 步骤：
    1. `which('gh')`检测安装
    2. `gh auth token`检测认证（stdout: 'ignore'，不读取token）
  - 超时：5秒

---

## 与其他文件的关系

- **依赖**：
  - `../which.ts` - 检测命令安装
  - `execa` - 执行gh命令
- **被依赖**：
  - 遥测系统收集用户环境信息

---

## 注意事项

- token通过stdout: 'ignore'避免进入进程内存
- 仅用于遥测，不用于实际API调用
- 网络无关（本地文件检查）
