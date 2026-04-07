# bridgeDebug.ts — 桥接调试与故障注入

> **一句话总结**：为Ant内部测试提供桥接故障注入和调试控制功能。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/bridge/bridgeDebug.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 136 |
| 主要职责 | 提供故障注入机制用于测试桥接恢复路径 |

---

## 功能概述

该文件实现了桥接(Bridge)功能的调试和故障注入机制，主要用于Ant内部测试。它允许开发者在运行时模拟各种桥接故障场景，以验证恢复逻辑的正确性。

该模块针对真实场景中常见的三种故障模式进行了设计：
- poll 404 not_found_error — 每周147K会话受影响
- WebSocket关闭(1002/1006) — 每周22K会话受影响  
- 注册瞬态失败 — 网络波动导致的重连问题

---

## 核心内容详解

### 导入与依赖

| 依赖 | 来源 | 用途 |
|------|------|------|
| `logForDebugging` | `../utils/debug.js` | 调试日志 |
| `BridgeFatalError` | `./bridgeApi.js` | 桥接致命错误类 |
| `BridgeApiClient` | `./types.js` | 桥接API客户端类型 |

### 类型定义

#### `BridgeFault`
故障注入配置对象
- `method`: 目标API方法（`'pollForWork' | 'registerBridgeEnvironment' | 'reconnectSession' | 'heartbeatWork'`）
- `kind`: 错误类型（`'fatal' | 'transient'`）
  - `fatal`: 致命错误，走 `handleErrorStatus → BridgeFatalError` 流程
  - `transient`: 瞬态错误，表现为axios拒绝（5xx/网络错误）
- `status`: HTTP状态码
- `errorType?`: 可选的错误类型字符串
- `count`: 剩余注入次数，消费后递减，到0时移除

#### `BridgeDebugHandle`
调试控制句柄接口
- `fireClose(code: number)`: 直接触发传输层的永久关闭处理器
- `forceReconnect()`: 调用 `reconnectEnvironmentWithSession()` 强制重连
- `injectFault(fault: BridgeFault)`: 为指定API方法排队故障
- `wakePollLoop()`: 中止容量等待，使故障立即生效
- `describe()`: 返回环境/会话ID描述信息

### 状态管理

| 变量 | 类型 | 说明 |
|------|------|------|
| `debugHandle` | `BridgeDebugHandle \| null` | 当前注册的调试句柄 |
| `faultQueue` | `BridgeFault[]` | 故障注入队列 |

### 主要函数

#### `registerBridgeDebugHandle`
- **类型**: `function`
- **用途**: 注册桥接调试句柄
- **参数**: `h: BridgeDebugHandle` - 调试句柄

#### `clearBridgeDebugHandle`
- **类型**: `function`
- **用途**: 清除调试句柄并清空故障队列

#### `getBridgeDebugHandle`
- **类型**: `function`
- **用途**: 获取当前调试句柄
- **返回值**: `BridgeDebugHandle | null`

#### `injectBridgeFault`
- **类型**: `function`
- **用途**: 向故障队列添加故障配置
- **参数**: `fault: BridgeFault` - 故障配置
- **副作用**: 记录调试日志

#### `wrapApiForFaultInjection`
- **类型**: `function`
- **用途**: 包装API客户端以支持故障注入
- **参数**: `api: BridgeApiClient` - 原始API客户端
- **返回值**: `BridgeApiClient` - 包装后的客户端
- **关键逻辑**:
  - 拦截四个核心API方法调用
  - 每次调用前检查故障队列
  - 匹配到故障时：
    - `fatal`类型：抛出 `BridgeFatalError`
    - `transient`类型：抛出普通 `Error`
  - 消费故障（count递减）
  - 无匹配故障时透传给真实API

---

## 设计要点

1. **模块级状态设计**: 每个REPL进程只有一个桥接，模块级状态便于 `/bridge-kick` 命令访问
2. **零开销外部构建**: 仅在 `USER_TYPE === 'ant'` 时调用包装函数
3. **故障消费机制**: 支持多次注入（count > 1），每次调用消费一次
4. **错误类型区分**: 明确区分fatal（触发销毁）和transient（触发重试）两种恢复路径

---

## 与其他文件的关系

- **依赖**:
  - `utils/debug.js` - 调试日志
  - `bridge/bridgeApi.js` - BridgeFatalError
  - `bridge/types.js` - BridgeApiClient类型
- **被依赖**: 由 `initBridgeCore` 在Ant模式下调用包装

---

## 注意事项

1. **仅限内部使用**: 该功能仅用于Ant内部测试，外部构建无此功能
2. **破坏性操作**: `fireClose` 和 `forceReconnect` 会中断正常连接
3. **副作用**: 故障注入有状态，测试后应调用 `clearBridgeDebugHandle` 清理
