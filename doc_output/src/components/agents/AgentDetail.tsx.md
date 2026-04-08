# AgentDetail.tsx — Agent详情展示组件

> **一句话总结**：展示单个Agent的完整详细信息，包括描述、工具、模型、系统提示等。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/AgentDetail.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 渲染Agent的完整信息视图 |

---

## 功能概述

`AgentDetail`组件用于显示单个Agent的完整详细信息。它以只读方式展示Agent的所有配置属性，包括：
- 文件路径（Agent定义文件的存储位置）
- 描述（WhenToUse）- 告诉Claude何时使用该Agent
- 工具列表 - Agent可以调用的工具
- 模型配置 - Agent使用的AI模型
- 权限模式 - Agent的权限设置
- 内存范围 - Agent的记忆能力
- Hooks和Skills - Agent的扩展能力
- 颜色 - Agent的视觉标识
- 系统提示 - 完整的系统提示词

组件支持键盘导航，按Enter或Esc可返回上级菜单。

---

## 核心内容详解

### 主要函数

- **AgentDetail**: 主组件函数
  - **参数**: `Props`对象，包含`agent`（Agent定义）、`tools`（可用工具）、`onBack`（返回回调）
  - **返回值**: React.ReactNode
  - **关键逻辑**: 
    - 使用`resolveAgentTools`解析Agent的工具配置
    - 使用`getAgentColor`获取Agent的颜色
    - 通过`useKeybinding`绑定Esc键返回

### 渲染逻辑

1. **文件路径**: 显示Agent定义文件的相对路径
2. **描述区域**: 展示`whenToUse`字段，说明Agent的使用场景
3. **工具列表**: 
   - 支持通配符（`*`）表示"所有工具"
   - 区分有效工具和无效工具（并显示警告）
4. **模型信息**: 显示Agent使用的模型
5. **可选字段**: 权限模式、内存范围、hooks、skills（仅当存在时显示）
6. **颜色预览**: 显示Agent的颜色标识
7. **系统提示**: 非内置Agent显示完整的系统提示词

---

## 设计要点

- 使用React Compiler缓存（`_c`函数）优化渲染性能
- 通过`useKeybinding`统一处理键盘交互
- 使用`Markdown`组件渲染系统提示词
- 支持内置Agent的特殊处理（不显示系统提示）

---

## 与其他文件的关系

- **依赖**: 
  - `resolveAgentTools` - 工具解析
  - `getAgentColor` - 颜色获取
  - `isBuiltInAgent` - 内置Agent判断
  - `Markdown` - 系统提示渲染
- **被依赖**: 
  - `AgentsMenu` - 作为详情视图使用

---

## 注意事项

- 内置Agent（built-in）不显示系统提示内容
- 工具列表中的无效工具会以警告形式显示
- 组件使用memo缓存策略，减少不必要的重渲染
