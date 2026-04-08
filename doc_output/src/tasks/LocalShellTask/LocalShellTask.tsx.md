# LocalShellTask.tsx — 本地Shell任务执行管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/LocalShellTask/LocalShellTask.tsx`
- **类型**: TSX 模块
- **主要功能**: 管理本地Shell命令的后台执行，支持前台/后台切换和卡顿检测

## 功能概述

本文件实现了LocalShellTask，用于处理bash命令的后台执行。支持命令的前台运行(可后续转为后台)、卡顿检测(识别交互式提示)、状态通知和任务终止。

## 核心内容详解

### 常量与配置

#### 卡顿检测配置
```typescript
const STALL_CHECK_INTERVAL_MS = 5_000      // 检查间隔：5秒
const STALL_THRESHOLD_MS = 45_000          // 卡顿阈值：45秒
const STALL_TAIL_BYTES = 1024              // 检查尾部字节数：1KB
```

#### 提示模式
用于识别命令是否阻塞等待键盘输入的正则表达式数组：
- `(y/n)`, `[y/n]`, `(yes/no)` - 是/否确认
- `Do you...`, `Would you...` - 定向问题
- `Press any key`, `Continue?`, `Overwrite?` - 继续提示

##### `looksLikePrompt`
检查文本尾部是否看起来像交互式提示。

### 主要函数

#### `startStallWatchdog`
启动卡顿监控狗：
- 检查输出文件大小变化
- 如果45秒内无增长且尾部看起来像提示，发送通知
- 返回取消函数
- Monitor任务(kind='monitor')跳过监控

通知内容包含：
- 任务ID和输出路径
- 提示检测到可能的交互式输入阻塞
- 建议：终止任务并使用管道输入或非交互式标志重运行

#### `enqueueShellNotification`
发送shell任务通知：
- 原子性检查并设置notified标志
- 中止活跃的推测(abortSpeculation)
- 根据kind和status生成不同的摘要
- 将通知加入消息队列

摘要变体：
- Monitor: "stream ended", "script failed", "stopped"
- Bash: "completed", "failed", "was stopped" (带BACKGROUND_BASH_SUMMARY_PREFIX)

#### `spawnShellTask`
生成并后台运行shell任务：
1. 使用提供的shellCommand.taskOutput.taskId作为任务ID
2. 注册清理处理器
3. 创建任务状态(isBackgrounded: true)
4. 注册任务到AppState
5. 调用shellCommand.background()转为后台运行
6. 启动卡顿监控狗
7. 设置结果处理器：
   - 取消监控狗
   - 刷新并清理shellCommand
   - 更新任务状态
   - 发送通知
   - 驱逐任务输出

#### `registerForeground`
注册前台任务(可后续后台化)：
- 创建任务状态(isBackgrounded: false)
- 注册任务到AppState
- 返回任务ID
- 用于运行时间较长的命令显示BackgroundHint后后台化

#### `backgroundTask`
将指定前台任务转为后台：
1. 验证任务存在、是LocalShellTask、未后台化、有shellCommand
2. 调用shellCommand.background()转为后台
3. 更新AppState设置isBackgrounded: true
4. 启动卡顿监控狗
5. 设置结果处理器(同spawnShellTask)

#### `hasForegroundTasks`
检查是否有可后台化的前台任务：
- 检查LocalShellTask：未后台化且有shellCommand
- 检查LocalAgentTask：未后台化且非main-session
- 用于决定Ctrl+B是后台化现有任务还是后台化会话

#### `backgroundAll`
后台化所有前台任务：
1. 获取所有前台bash任务ID
2. 逐个调用backgroundTask
3. 后台化所有前台代理任务(通过backgroundAgentTask)
4. 取消当前会话的推测(abortSpeculation)

### LocalShellTask 对象

实现Task接口：
```typescript
export const LocalShellTask: Task = {
  name: 'LocalShellTask',
  type: 'local_bash',
  async kill(taskId, setAppState) {
    killTask(taskId, setAppState)  // 委托给killShellTasks.ts
  }
}
```

## 设计要点

1. **卡顿检测**：识别交互式提示，避免慢命令误报
2. **前台/后台切换**：支持命令先前台运行，后转为后台
3. **TaskOutput集成**：数据自动流经TaskOutput，无需流监听器
4. **代理关联**：agentId用于代理退出时清理孤儿任务
5. **Monitor变体**：kind='monitor'提供不同的UI表现

## 与其他文件的关系

- **导入**：
  - bun:bundle的feature标志
  - XML常量、AppState、Task接口
  - AgentId、ShellCommand类型
  - 各种工具函数(清理注册、任务框架等)
  - LocalAgentTask的backgroundAgentTask
  - guards.ts和killShellTasks.ts

- **被调用者**：
  - BashTool - 创建shell任务
  - 快捷键处理 - Ctrl+B后台化

## 注意事项

1. **卡顿检测门控**：仅当尾部匹配PROMPT_PATTERNS时才发送通知，避免慢命令误报
2. **通知去重**：原子性检查notified标志防止重复通知
3. **推测中止**：后台任务状态变化时中止推测，避免结果引用过期输出
4. **优先级**：Monitor通知使用'next'优先级，bash使用'later'
