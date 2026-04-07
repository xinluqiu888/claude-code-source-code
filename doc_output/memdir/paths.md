# paths.ts — 内存路径管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/paths.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

管理内存目录的路径解析、验证和访问。支持自动内存启用检查、路径覆盖和项目范围的内存组织。

## 核心内容详解

### 主要导出

1. **isAutoMemoryEnabled()** — 检查自动内存是否启用
   优先级链 (第一个定义的值获胜):
   1. `CLAUDE_CODE_DISABLE_AUTO_MEMORY` 环境变量 (1/true → OFF, 0/false → ON)
   2. `CLAUDE_CODE_SIMPLE` (--bare) → OFF
   3. CCR 无持久存储 → OFF (无 `CLAUDE_CODE_REMOTE_MEMORY_DIR`)
   4. settings.json 中的 `autoMemoryEnabled`
   5. 默认: enabled

2. **isExtractModeActive()** — 检查提取模式是否激活
   - 需要 `tengu_passport_quail` 特性门控
   - 非交互式会话需要额外特性

3. **getMemoryBaseDir()** — 获取内存基础目录
   - 优先使用 `CLAUDE_CODE_REMOTE_MEMORY_DIR` 环境变量
   - 默认使用 `~/.claude`

4. **getAutoMemPath()** — 获取自动内存目录路径
   - 记忆化函数 (memoized)
   - 解析顺序:
     1. `CLAUDE_COWORK_MEMORY_PATH_OVERRIDE` 环境变量
     2. settings.json 中的 `autoMemoryDirectory`
     3. `<memoryBase>/projects/<sanitized-git-root>/memory/`
   - 键控于 projectRoot

5. **getAutoMemDailyLogPath(date?)** — 获取每日日志文件路径
   - 格式: `<autoMemPath>/logs/YYYY/MM/YYYY-MM-DD.md`
   - 用于助手模式 (KAIROS)

6. **getAutoMemEntrypoint()** — 获取自动内存入口点
   - 返回 `MEMORY.md` 完整路径

7. **isAutoMemPath(absolutePath)** — 检查路径是否在自动内存目录内
   - 安全: 规范化以防止路径遍历绕过

### 路径验证

**validateMemoryPath(raw, expandTilde)**:
- 安全检查危险路径
- 拒绝相对路径、根/近根路径、Windows 驱动器根、UNC 路径、空字节
- 支持 `~` 展开
- 返回规范化的路径或 undefined

### 安全考量

1. **路径遍历防护**:
   - 规范化路径 (`normalize()`)
   - 检查前缀匹配
   - 防止 `..` 段逃逸

2. **路径注入防护**:
   - 拒绝空字节
   - 拒绝 URL 编码遍历 (`%2e%2e%2f`)
   - 拒绝 Unicode 规范化攻击 (全角字符)

3. **设置源限制**:
   - `projectSettings` 被排除在路径覆盖外
   - 防止恶意仓库设置危险路径

## 设计要点

1. **记忆化**:
   - `getAutoMemPath` 使用 lodash memoize
   - 键控于 projectRoot
   - 测试中可清除缓存

2. **路径解析顺序**:
   - 环境变量优先
   - 设置次之
   - 默认最后

3. **Git 仓库共享**:
   - 使用 `findCanonicalGitRoot` 确保所有工作树共享同一内存目录
   - 通过规范化 git 根目录实现

4. **NFC 规范化**:
   - 所有返回路径使用 NFC Unicode 规范化
   - 确保 macOS 和 Linux 间的一致性

5. **尾随分隔符**:
   - 所有目录路径以分隔符结尾
   - 便于前缀匹配

## 与其他文件的关系

- **../bootstrap/state.js**: getProjectRoot, getOriginalCwd
- **../services/analytics/growthbook.js**: 特性门控
- **../utils/git.js**: findCanonicalGitRoot
- **../utils/path.js**: sanitizePath

## 注意事项

1. 路径验证拒绝危险模式但不验证存在性
2. 环境变量覆盖不支持 `~` 展开
3. settings.json 路径支持 `~` 展开但受信任源限制
4. CCR 环境需要显式内存目录设置
5. 记忆化缓存需要在测试中清除
