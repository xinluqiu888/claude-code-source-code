# pillLabel.ts — 后台任务指示器标签生成

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tasks/pillLabel.ts`
- **类型**: TypeScript 模块
- **主要功能**: 生成后台任务集的紧凑footer pill标签

## 功能概述

本文件提供了生成后台任务指示器标签的函数，用于footer pill和turn-duration转录行，确保两个界面使用一致的术语。

## 核心内容详解

### 主要函数

#### `getPillLabel`
生成后台任务集的紧凑标签：

参数：
- tasks: BackgroundTaskState[] - 后台任务数组

逻辑流程：

1. **单类型任务处理** (所有任务类型相同)：
   
   **local_bash**：
   - 统计monitor和shell数量
   - 分别显示："1 shell", "2 shells", "1 monitor", "2 monitors"
   - 多个部分用逗号分隔
   
   **in_process_teammate**：
   - 统计不同teamName的数量
   - 显示："1 team"或"N teams"
   
   **local_agent**：
   - 显示："1 local agent"或"N local agents"
   
   **remote_agent**：
   - 如果是ultraplan：
     - `plan_ready`: ◆ ultraplan ready
     - `needs_input`: ◇ ultraplan needs your input
     - 其他: ◇ ultraplan
   - 其他：显示"◇ 1 cloud session"或"◇ N cloud sessions"
   
   **local_workflow**：
   - 显示："1 background workflow"或"N background workflows"
   
   **monitor_mcp**：
   - 显示："1 monitor"或"N monitors"
   
   **dream**：
   - 显示："dreaming"

2. **混合类型任务**：
   - 显示："N background task"或"N background tasks"

#### `pillNeedsCta`
判断pill是否应显示"· ↓ to view"行动召唤：

返回true条件：
1. 只有1个任务
2. 类型为remote_agent
3. isUltraplan为true
4. ultraplanPhase已定义

设计原理：
- 只有needs_input和plan_ready两种注意力状态显示CTA
- 普通running状态只显示diamond + label

## 设计要点

1. **一致术语**：footer pill和turn-duration转录行使用相同标签
2. **类型感知**：根据任务类型显示不同的标签格式
3. **Ultraplan状态**：使用钻石符号(◇ ◆)区分ultraplan阶段
4. **数量复数**：正确处理单复数形式(1 shell vs 2 shells)
5. **CTA门控**：仅在需要用户注意的状态显示行动召唤

## 与其他文件的关系

- **导入**：
  - `DIAMOND_FILLED`, `DIAMOND_OPEN` from constants/figures - 钻石符号
  - `count` from utils/array - 数组计数
  - `BackgroundTaskState` from ./types - 后台任务状态类型

- **被调用者**：
  - Footer组件 - 显示后台任务pill
  - Turn-duration组件 - 显示转录行
  - 后台任务指示器相关UI

## 注意事项

1. **钻石符号**：◇ (空心)表示运行/需要输入，◆ (实心)表示计划就绪
2. **Monitor分离**：local_bash任务中monitor和shell分别计数显示
3. **团队统计**：in_process_teammate按teamName去重计数
4. **回退处理**：混合类型任务使用通用"background task"标签
5. **CTA条件**：严格的单任务+ultraplan+有相条件限制CTA显示
