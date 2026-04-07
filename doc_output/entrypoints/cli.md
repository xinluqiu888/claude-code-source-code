# cli.tsx — CLI 启动入口

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/cli.tsx`
- **类型**: CLI 启动入口文件
- **语言**: TypeScript

## 功能概述

Claude Code CLI 的引导入口点。在加载完整 CLI 之前检查特殊标志，使用动态导入以最小化模块评估，实现快速路径优化。`--version` 快速路径除了此文件外零导入。

## 核心内容详解

### 环境初始化

1. **Corepack 修复**:
   - 设置 `COREPACK_ENABLE_AUTO_PIN = '0'`
   - 防止 yarnpkg 自动添加到 package.json

2. **CCR 内存配置**:
   - 在 CCR 环境 (CLAUDE_CODE_REMOTE) 中设置最大堆大小为 8GB
   - 容器通常有 16GB 内存

3. **Ablated Baseline 配置**:
   - 支持 `CLAUDE_CODE_ABLATION_BASELINE` 环境变量
   - 自动禁用多种高级功能

### 快速路径处理

按优先级顺序处理各种快速路径：

1. **--version/-v**: 零模块加载，直接输出版本
2. **--dump-system-prompt**: 输出渲染的系统提示并退出
3. **--claude-in-chrome-mcp**: Chrome MCP 服务器
4. **--chrome-native-host**: Chrome Native Host
5. **--computer-use-mcp**: 计算机使用 MCP (CHICAGO_MCP 特性)
6. **--daemon-worker**: 守护进程工作流
7. **remote-control/rc/remote/sync/bridge**: 远程控制/桥接模式
8. **daemon**: 守护进程主进程
9. **ps/logs/attach/kill --bg/--background**: 后台会话管理
10. **new/list/reply**: 模板作业命令
11. **environment-runner**: BYOC 环境运行器
12. **self-hosted-runner**: 自托管运行器
13. **--worktree --tmux**: Tmux 工作树快速路径

### 标志重定向

- `--update` / `--upgrade` 重定向到 `update` 子命令
- `--bare` 设置 `CLAUDE_CODE_SIMPLE` 环境变量

### 主流程

1. 启动早期输入捕获
2. 导入并运行主 CLI
3. 记录启动性能检查点

## 设计要点

1. **动态导入策略**:
   - 所有非核心导入使用动态 import
   - 每个快速路径仅加载所需模块
   - 最小化启动时间和内存占用

2. **特性门控内联**:
   - `feature()` 调用保持内联用于构建时死代码消除
   - 运行时检查使用单独的函数

3. **性能分析**:
   - 使用 `profileCheckpoint` 记录启动阶段
   - 帮助识别性能瓶颈

4. **错误处理**:
   - 桥接模式检查认证和版本
   - 策略限制检查
   - 优雅的错误退出

5. **环境准备**:
   - 早期环境变量设置
   - 代理和 mTLS 配置
   - 工作树模式检测

## 与其他文件的关系

- **../main.js**: 完整 CLI 主入口
- **../utils/startupProfiler.js**: 启动性能分析
- **../utils/config.js**: 配置系统
- **../bridge/**: 桥接模式相关模块
- **../daemon/**: 守护进程相关模块
- **../cli/bg.js**: 后台会话管理

## 注意事项

1. 快速路径顺序很重要，特定检查必须在其他检查之前
2. `--daemon-worker` 必须在 `daemon` 子命令检查之前
3. `--bare` 标志需要早期设置以影响后续模块评估
4. 特性门控在构建时进行死代码消除
5. 远程控制功能需要 OAuth 令牌
6. Tmux 工作树快速路径需要单独处理
