# LocalMainSessionTask.ts — 主会话后台任务管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/LocalMainSessionTask.ts`
- **类型**: TypeScript 模块
- **主要功能**: 处理主会话查询的后台化，支持将当前会话转为后台运行

## 功能概述

本文件实现了LocalMainSessionTask，当用户按Ctrl+B两次时会话被"后台化"：查询继续在后台运行，UI清空到新的提示，完成时发送通知。复用LocalAgentTask的状态结构，因为行为相似。

## 核心内容详解

### 类型定义

#### `LocalMainSessionTaskState`
主会话任务状态(继承LocalAgentTaskState)：
```typescript
type LocalMainSessionTaskState = LocalAgentTaskState & {
  agentType: 'main-session'
}
```

### 常量

#### `DEFAULT_MAIN_SESSION_AGENT`
主会话任务的默认代理定义：
- agentType: 'main-session'
- whenToUse: 'Main session query'
- source: 'userSettings'
- getSystemPrompt: 返回空字符串

### 主要函数

#### `generateMainSessionTaskId`
生成主会话任务唯一ID：
- 使用's'前缀(区别于代理任务的'a'前缀)
- 8位随机字符(数字+小写字母)

#### `registerMainSessionTask`
注册后台化的主会话任务：

参数：
- description: 任务描述
- setAppState: 状态设置函数
- mainThreadAgentDefinition: 可选代理定义(运行--agent时)
- existingAbortController: 可选现有中止控制器(用于后台化活动查询)

执行流程：
1. 生成任务ID
2. 初始化任务输出符号链接到隔离的转录文件
3. 使用现有或新的中止控制器
4. 注册清理处理器
5. 创建任务状态(isBackgrounded: true)
6. 注册任务到AppState
7. 验证任务注册成功
8. 返回taskId和abortSignal

关键设计：
- 使用隔离的转录路径(非getTranscriptPath)
- 避免/clear后的后台查询损坏对话
- 符号链接在clearConversation时重新链接

#### `completeMainSessionTask`
完成主会话任务并发送通知：

执行流程：
1. 检查任务是否仍在运行
2. 记录wasBackgrounded状态(决定是否发送通知)
3. 清理unregisterCleanup
4. 更新任务状态(completed/failed)
5. 驱逐任务输出
6. 如果仍后台化，发送通知；否则设置notified标志并发送SDK事件

通知决策：
- 后台化任务：发送完整通知
- 前台化任务：仅设置notified标志，发送SDK终止事件

#### `enqueueMainSessionNotification`
发送后台会话完成通知：
- 原子性检查并设置notified标志
- 构建包含任务ID、输出路径、状态、摘要的XML通知
- 排入消息队列

#### `foregroundMainSessionTask`
前台化主会话任务：
1. 获取任务消息
2. 恢复之前前台化的任务到后台(如果存在)
3. 设置新前台任务
4. 返回任务的消息数组

状态转换：
- 前一个前台任务(如果存在且是local_agent)：isBackgrounded设为true
- 新任务：isBackgrounded设为false

#### `isMainSessionTask`
检查任务是否为主会话任务(vs普通代理任务)：
- 检查type === 'local_agent'
- 检查agentType === 'main-session'

#### `startBackgroundSession`
启动新的后台会话：

参数：
- messages: 初始消息数组
- queryParams: 查询参数(除messages外)
- description: 描述
- setAppState: 状态设置函数
- agentDefinition: 可选代理定义

执行流程：
1. 注册主会话任务
2. 持久化预后台化对话到任务转录
3. 包装在agentContext中运行
4. 启动query()流
5. 处理消息流：
   - 检查中止信号
   - 推送消息到bgMessages
   - 写入转录
   - 统计token和工具使用
   - 跟踪最近活动
   - 更新AppState进度
6. 完成或失败时调用completeMainSessionTask

代理上下文：
- agentId设为taskId
- 支持clearInvokedSkills选择性保留
- AsyncLocalStorage隔离并发异步链

## 设计要点

1. **隔离转录**：使用独立转录路径，避免/clear后后台查询损坏主对话
2. **复用LocalAgent状态**：主会话任务复用LocalAgentTaskState结构
3. **前台/后台切换**：支持任务在后台运行后前台化查看
4. **Agent上下文**：支持技能调用作用域隔离和/clear保留
5. **增量写入**：每消息写入转录，支持实时TaskOutput进度

## 与其他文件的关系

- **导入**：
  - UUID类型、randomBytes
  - XML常量、query函数、token估算服务
  - Task接口、AgentDefinition
  - 各种工具函数(中止控制器、agent上下文、任务框架等)

- **被调用者**：
  - 快捷键处理 - Ctrl+B后台化会话
  - 会话管理 - 前台化任务

## 注意事项

1. **ID前缀**：'s'前缀区别于普通代理任务('a'前缀)
2. **中止控制器复用**：后台化活动查询时复用现有中止控制器
3. **通知抑制**：前台化任务不发送XML通知(TUI用户正在观看)
4. **消息限制**：完成时只保留最后一条消息到任务状态
5. **活动跟踪**：MAX_RECENT_ACTIVITIES=5限制最近活动数量
