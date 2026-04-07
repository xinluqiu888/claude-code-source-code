# bridgePermissionCallbacks.ts — 桥接权限回调类型

> **一句话总结**：定义桥接权限请求/响应的回调接口和类型守卫。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/bridge/bridgePermissionCallbacks.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 44 |
| 主要职责 | 桥接权限回调的类型定义和验证 |

---

## 功能概述

该文件定义了Remote Control桥接功能中处理权限请求和响应所需的类型定义。它提供了类型安全的回调接口，用于在桥接会话中处理工具权限请求的发送、响应和取消操作。

---

## 核心内容详解

### 导入与依赖

| 依赖 | 来源 | 用途 |
|------|------|------|
| `PermissionUpdate` | `../utils/permissions/PermissionUpdateSchema.js` | 权限更新数据类型 |

### 类型定义

#### `BridgePermissionResponse`
权限响应对象
- `behavior`: `'allow' \| 'deny'` - 权限决定（允许/拒绝）
- `updatedInput?`: `Record<string, unknown>` - 修改后的输入参数
- `updatedPermissions?`: `PermissionUpdate[]` - 权限更新列表
- `message?`: `string` - 可选的消息说明

#### `BridgePermissionCallbacks`
权限回调接口
- **`sendRequest`**:
  - 参数: `requestId`, `toolName`, `input`, `toolUseId`, `description`, `permissionSuggestions?`, `blockedPath?`
  - 用途: 发送权限请求到web应用
  
- **`sendResponse`**:
  - 参数: `requestId`, `response`
  - 用途: 发送权限响应

- **`cancelRequest`**:
  - 参数: `requestId`
  - 用途: 取消待处理的控制请求，使web应用可关闭提示

- **`onResponse`**:
  - 参数: `requestId`, `handler`
  - 返回值: 取消订阅函数
  - 用途: 注册响应处理器，返回函数用于取消订阅

### 类型守卫函数

#### `isBridgePermissionResponse`
- **类型**: `function`
- **用途**: 验证未知值是否为有效的 `BridgePermissionResponse`
- **参数**: `value: unknown`
- **返回值**: `boolean`（类型谓词 `value is BridgePermissionResponse`）
- **验证逻辑**:
  1. 检查值是否存在且为对象类型
  2. 检查是否存在 `behavior` 属性
  3. 验证 `behavior` 值为 `'allow'` 或 `'deny'`

---

## 设计要点

1. **类型安全**: 使用类型谓词而非不安全的类型断言
2. **最小验证**: 仅验证必要的判别式属性（`behavior`）
3. **可扩展性**: 响应对象设计为可选属性丰富，核心判别式固定

---

## 与其他文件的关系

- **依赖**:
  - `utils/permissions/PermissionUpdateSchema.js` - 权限更新类型
- **被依赖**: 由桥接核心实现和权限处理模块使用

---

## 注意事项

1. **轻量级设计**: 本文件仅包含类型定义，无运行时逻辑
2. **验证严格性**: 类型守卫仅检查最小必要属性，调用方需进一步验证
