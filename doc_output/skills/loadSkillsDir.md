# loadSkillsDir.ts — 技能目录加载器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/skills/loadSkillsDir.ts`
- **类型**: TypeScript 模块
- **作用**: 从文件系统加载和管理技能（skills/commands）

## 功能概述

本模块是Claude Code技能系统的核心，负责从多个来源加载技能：
- 托管策略目录（managed policy settings）
- 用户配置目录
- 项目目录（向上遍历到home目录）
- 额外的显式指定目录（--add-dir）
- 遗留的/commands/目录格式

同时支持动态技能发现、条件技能激活和技能去重。

## 核心内容详解

### 主要导出函数

#### `getSkillDirCommands(cwd: string): Promise<Command[]>`
使用memoize缓存，从所有来源加载技能命令。

**加载优先级**（由后到前，后面的覆盖前面的）：
1. 托管策略技能（policySettings）
2. 用户设置技能（userSettings）
3. 项目设置技能（projectSettings）
4. 额外目录技能（additionalDirs）
5. 遗留commands目录技能

#### `createSkillCommand(options): Command`
从解析的数据创建技能命令对象，处理：
- 参数替换（`${argName}` → 实际值）
- 变量替换（`${CLAUDE_SKILL_DIR}`, `${CLAUDE_SESSION_ID}`）
- Shell命令执行（`!`command`` 和 ```! ... ``` 代码块）
- 安全性控制（MCP技能不执行shell命令）

#### `parseSkillFrontmatterFields(frontmatter, content, name)`
解析技能前置元数据字段，返回标准化的技能配置对象。

### 技能类型支持

**目录格式**（/skills/和/commands/）：
```
skill-name/
  └── SKILL.md
```

**遗留文件格式**（仅/commands/）：
```
command-name.md
```

### 动态技能发现

- `discoverSkillDirsForPaths(filePaths, cwd)`: 从文件路径向上遍历发现技能目录
- `addSkillDirectories(dirs)`: 加载发现的技能目录
- `getDynamicSkills()`: 获取所有动态发现的技能

### 条件技能

技能可以带有`paths`前置元数据，只在匹配特定文件时才激活：

```yaml
---
paths:
  - "src/**/*.ts"
---
```

激活流程：
1. 加载时存储在`conditionalSkills`映射中
2. 文件操作时检查路径匹配
3. 匹配后移动到`dynamicSkills`映射中可用

### 技能去重

使用`realpath`解析符号链接，通过规范路径去重：
- 相同文件通过不同路径访问（符号链接、重叠父目录）
- 优先保留先加载的副本
- 记录跳过的重复项用于调试

## 设计要点

1. **缓存策略**: 使用lodash的memoize缓存技能加载结果
2. **异步索引构建**: `loadFromFileListAsync`分块处理，每4ms让出事件循环
3. **Top-K搜索优化**: 维护排序的top-k数组，避免O(n log n)的全排序
4. **智能大小写**: 小写查询→大小写不敏感；含大写→大小写敏感
5. **测试文件惩罚**: 包含"test"的路径获得1.05x惩罚（上限1.0）

## 与其他文件的关系

- **导出到 mcpSkillBuilders.ts**: 注册`createSkillCommand`和`parseSkillFrontmatterFields`
- **被 commands.ts 导入**: 静态导入触发初始化
- **使用 frontmatterParser.ts**: 解析前置元数据
- **使用 markdownConfigLoader.ts**: 加载遗留commands目录

## 注意事项

1. **bare模式**: `--bare`标志跳过自动发现，只加载显式--add-dir路径
2. **安全策略**: `skillsLocked`（plugin-only策略）时跳过技能和commands目录
3. **Git忽略检查**: 动态发现的技能目录如果在git忽略中会被跳过
4. **内存管理**: 清除缓存时同时清除技能加载缓存和Markdown文件缓存

### 性能优化

- 使用位图（bitmap）进行O(1)路径字符存在性检查
- 预计算小写路径和字符位图
- Top-K搜索避免全量排序
- 异步分块构建避免阻塞主线程
