# registry.ts — 命令规格注册表

> **一句话总结**：管理命令规格（CommandSpec），提供内置spec和动态加载Fig autocomplete spec的功能，用于命令补全和参数分析。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/registry.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~54行 |
| 主要职责 | 命令规格管理和动态加载 |

---

## 功能概述

该模块是Claude Code的命令规格注册中心。它维护了两层命令规格：
1. **内置Specs**：项目内置的常用命令规格（如pyright、timeout等）
2. **动态Specs**：通过Fig Autocomplete库动态加载的规格

CommandSpec定义了命令的结构信息，包括子命令、参数、选项等，用于：
- 命令补全提示
- 权限系统的前缀提取
- 识别包装命令和危险参数

---

## 核心内容详解

### 主要导出类型

- **`CommandSpec`**
  - 类型：接口
  - 属性：
    - `name`: string - 命令名称
    - `description?`: string - 命令描述
    - `subcommands?`: CommandSpec[] - 子命令列表
    - `args?`: Argument | Argument[] - 参数定义
    - `options?`: Option[] - 选项定义

- **`Argument`**
  - 类型：接口
  - 属性：
    - `name?`: string - 参数名
    - `description?`: string - 描述
    - `isDangerous?`: boolean - 是否危险参数
    - `isVariadic?`: boolean - 是否可变参数
    - `isOptional?`: boolean - 是否可选
    - `isCommand?`: boolean - 是否为命令参数（如sudo后的命令）
    - `isModule?`: string | boolean - 模块参数（如python -m）
    - `isScript?`: boolean - 脚本文件参数（如node script.js）

- **`Option`**
  - 类型：接口
  - 属性：
    - `name`: string | string[] - 选项名（可短可长）
    - `description?`: string - 描述
    - `args?`: Argument | Argument[] - 选项参数
    - `isRequired?`: boolean - 是否必需

### 主要导出函数

- **`loadFigSpec`**
  - 参数：`command`（string）- 命令名
  - 返回值：`Promise<CommandSpec | null>`
  - 用途：从@withfig/autocomplete动态加载spec
  - 安全检查：拒绝包含路径分隔符或..的命令名

- **`getCommandSpec`**
  - 参数：`command`（string）- 命令名
  - 返回值：`Promise<CommandSpec | null>`
  - 用途：获取命令的完整spec（内置+动态）
  - 特性：使用LRU缓存（memoizeWithLRU）

---

## 设计要点

1. **双层查找**：先查内置specs，未找到再动态加载Fig spec
2. **LRU缓存**：常用命令的spec会被缓存以提高性能
3. **安全加载**：动态加载时验证命令名，防止路径遍历
4. **延迟加载**：Fig specs按需加载，减少启动时间

---

## 与其他文件的关系

- **依赖**：
  - `../memoize.ts` - LRU缓存实现
  - `./specs/index.ts` - 内置spec集合
- **被依赖**：
  - `./prefix.ts` - 命令前缀提取
  - `../shell/specPrefix.ts` - 前缀构建
  - `../permissions/permissions.ts` - 权限验证

---

## 注意事项

- Fig specs来自@withfig/autocomplete包，需要单独安装
- 内置specs优先级高于Fig specs
- 缓存键为命令名字符串，大小写敏感
- 加载失败（如命令不存在）会返回null而不会抛出错误
