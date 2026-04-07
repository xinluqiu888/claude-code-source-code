# prompt.ts — 伙伴提示词生成

> **一句话总结**：为模型生成伙伴系统的上下文提示词和附件。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/buddy/prompt.ts` |
| 文件类型 | TypeScript |
| 代码行数 | 37 |
| 主要职责 | 生成伙伴介绍文本和消息附件 |

---

## 功能概述

该文件负责为AI模型生成关于伙伴系统的上下文提示词。当用户拥有伙伴时，它会向模型说明伙伴的角色定位——一个独立的小观察者，用户可以直接与其对话，而模型应保持简洁回应。

---

## 核心内容详解

### 导入与依赖

| 依赖 | 来源 | 用途 |
|------|------|------|
| `feature` | `bun:bundle` | 编译时特性检查 |
| `Message` | `../types/message.js` | 消息类型 |
| `Attachment` | `../utils/attachments.js` | 附件类型 |
| `getGlobalConfig` | `../utils/config.js` | 配置读取 |
| `getCompanion` | `./companion.js` | 获取伙伴信息 |

### 核心函数

#### `companionIntroText`
- **参数**: 
  - `name`: `string` - 伙伴名字
  - `species`: `string` - 伙伴物种
- **返回值**: `string` - Markdown格式的提示词
- **内容结构**:
  ```markdown
  # Companion
  
  A small {species} named {name} sits beside the user's input box...
  ```
- **关键指令**:
  - 说明伙伴是独立观察者，不是AI本身
  - 用户直接称呼伙伴名字时，AI应简短回应或只回答给自己的部分
  - 不要解释"我不是{name}"
  - 不要替伙伴描述可能说的话

#### `getCompanionIntroAttachment`
- **参数**: 
  - `messages`: `Message[] \| undefined` - 历史消息
- **返回值**: `Attachment[]` - 附件数组（空或单元素）
- **逻辑流程**:
  1. 检查 `BUDDY` 编译特性是否启用
  2. 获取当前伙伴，检查是否静音
  3. 检查历史消息中是否已存在同名伙伴的介绍
  4. 返回 `companion_intro` 类型附件

### 附件结构

```typescript
{
  type: 'companion_intro',
  name: string,      // 伙伴名字
  species: string,   // 伙伴物种
}
```

### 去重逻辑

函数会遍历历史消息，检查是否已存在：
- `type === 'attachment'`
- `attachment.type === 'companion_intro'`
- `attachment.name === companion.name`

满足条件则返回空数组，避免重复介绍。

---

## 设计要点

1. **特性门控**: 通过 `feature('BUDDY')` 编译时控制功能开关
2. **用户控制**: 尊重 `companionMuted` 配置
3. **会话级去重**: 每个会话只介绍一次同名的伙伴
4. **简洁提示**: 明确伙伴角色边界，避免AI混淆

---

## 与其他文件的关系

- **依赖**:
  - `types.ts` - 类型定义
  - `companion.ts` - 伙伴获取
  - `config.ts` - 配置读取
- **被依赖**: 由消息处理系统调用，附加到模型请求

---

## 注意事项

1. **编译时特性**: BUDDY特性必须在构建时启用
2. **静音尊重**: 用户可通过配置完全禁用伙伴功能
3. **模型指令**: 提示词明确约束AI在伙伴被直接称呼时的行为
