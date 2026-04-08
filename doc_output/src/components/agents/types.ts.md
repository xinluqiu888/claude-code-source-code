# types.ts — Agent类型定义

> **一句话总结**：定义Agent相关的类型和常量。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/types.ts` |
| 文件类型 | TypeScript类型定义文件 |
| 主要职责 | 集中定义Agent模块使用的类型和常量 |

---

## 功能概述

`types.ts`集中定义了Agent模块使用的核心类型，包括：
- Agent存储路径常量
- 模式状态类型（ModeState）
- 验证结果类型

---

## 核心内容详解

### 常量

- **AGENT_PATHS**: Agent存储路径
  - `FOLDER_NAME`: '.claude' - 配置文件夹名
  - `AGENTS_DIR`: 'agents' - Agent子目录名

### 类型定义

- **WithPreviousMode**: 包含previousMode字段的混入类型
- **WithAgent**: 包含agent字段的混入类型

- **ModeState**: 模式状态联合类型
  - `{ mode: 'main-menu' }`: 主菜单
  - `{ mode: 'list-agents', source: SettingSource | 'all' | 'built-in' }`: 列表视图
  - `{ mode: 'agent-menu', agent, previousMode }`: Agent菜单
  - `{ mode: 'view-agent', agent, previousMode }`: 查看Agent
  - `{ mode: 'create-agent' }`: 创建Agent
  - `{ mode: 'edit-agent', agent, previousMode }`: 编辑Agent
  - `{ mode: 'delete-confirm', agent, previousMode }`: 删除确认

- **AgentValidationResult**: 验证结果
  - `isValid: boolean`: 是否有效
  - `warnings: string[]`: 警告列表
  - `errors: string[]`: 错误列表

---

## 设计要点

- 使用交集类型构建复杂状态类型
- 集中管理路径常量，避免硬编码
- 验证结果分离错误和警告

---

## 与其他文件的关系

- **依赖**:
  - `SettingSource` - 设置来源类型
  - `AgentDefinition` - Agent定义类型
- **被依赖**:
  - `AgentsMenu` - 使用ModeState
  - `validateAgent.ts` - 使用AgentValidationResult

---

## 注意事项

- ModeState使用联合类型实现状态机
- 每个模式可以包含额外的上下文数据
- 路径常量与实际的文件系统结构保持一致
