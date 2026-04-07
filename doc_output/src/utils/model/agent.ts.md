# agent.ts — 代理模型管理器

> **一句话总结**：管理子代理（subagent）的模型选择和继承逻辑，支持Bedrock跨区域推理前缀传递。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/model/agent.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~158行 |
| 主要职责 | 子代理模型选择和配置 |

---

## 功能概述

该模块处理Claude Code子代理的模型选择：
1. 支持模型别名（sonnet/opus/haiku/inherit）
2. 继承父级模型的Bedrock跨区域前缀
3. 根据订阅级别提供不同的默认选项
4. 处理工具指定的模型覆盖

---

## 核心内容详解

### 导出函数

- **`getAgentModel`**
  - 参数：agentModel, parentModel, toolSpecifiedModel, permissionMode
  - 用途：计算子代理应使用的有效模型
  - 逻辑：
    - 优先使用CLAUDE_CODE_SUBAGENT_MODEL环境变量
    - 工具指定模型次之
    - 'inherit'继承父模型（运行时解析）
    - 处理Bedrock区域前缀继承

- **`getAgentModelDisplay`**
  - 用途：生成模型显示的友好文本

- **`getAgentModelOptions`**
  - 用途：返回可用的代理模型选项

### 辅助函数

- **`aliasMatchesParentTier`**
  - 用途：检查别名是否与父模型同系列（如都是opus）

---

## 与其他文件的关系

- **依赖**：
  - `./aliases.ts` - 模型别名
  - `./bedrock.ts` - 区域前缀处理
  - `./model.ts` - 模型解析
  - `../permissions/PermissionMode.ts` - 权限模式
- **被依赖**：
  - Agent创建和执行流程

---

## 注意事项

- Bedrock跨区域前缀继承对IAM权限很重要
- 'inherit'模型在plan模式下有特殊处理（opusplan）
