# teamMemPaths.ts — 团队内存路径管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/memdir/teamMemPaths.ts`
- **类型**: TypeScript 模块
- **语言**: TypeScript

## 功能概述

管理团队内存的路径验证、安全检查和访问控制。团队内存是自动内存的子目录，支持多用户间共享内存。

## 核心内容详解

### 错误类型

**PathTraversalError** — 路径遍历错误
- 检测和抛出路径遍历尝试
- 包含描述性错误消息

### 主要导出

1. **isTeamMemoryEnabled()** — 检查团队内存是否启用
   - 需要自动内存已启用
   - 需要 `tengu_herring_clock` 特性门控

2. **getTeamMemPath()** — 获取团队内存路径
   - 格式: `<autoMemPath>/team/`
   - NFC Unicode 规范化

3. **getTeamMemEntrypoint()** — 获取团队内存入口点
   - 返回 `team/MEMORY.md` 路径

4. **isTeamMemPath(filePath)** — 检查路径是否在团队内存目录内
   - 使用 path.resolve() 规范化
   - 防止 `..` 段遍历

5. **validateTeamMemWritePath(filePath)** — 验证写入路径
   - 检查空字节
   - 第一遍: 规范化并检查字符串级包含
   - 第二遍: 解析最深现有祖先的符号链接
   - 抛出 PathTraversalError 如果无效

6. **validateTeamMemKey(relativeKey)** — 验证相对键
   - 消毒路径键 (sanitizePathKey)
   - 检查 URL 编码遍历
   - 检查 Unicode 规范化攻击
   - 解析符号链接
   - 返回解析后的绝对路径

7. **isTeamMemFile(filePath)** — 检查文件路径
   - 综合检查: 团队内存启用 + 路径在目录内

### 路径消毒

**sanitizePathKey(key)**:
- 检查空字节
- 检查 URL 编码遍历 (`%2e%2e%2f` = `../`)
- 检查 Unicode 规范化攻击 (全角字符)
- 拒绝反斜杠 (Windows 路径分隔符)
- 拒绝绝对路径

### 符号链接解析

**realpathDeepestExisting(absolutePath)**:
- 解析最深现有祖先的符号链接
- 处理 ENOENT (不存在)
- 处理 ENOTDIR (非目录)
- 检测悬挂符号链接 (攻击向量)
- 检测符号链接循环

**isRealPathWithinTeamDir(realCandidate)**:
- 比较真实路径与真实团队目录
- 前缀攻击保护 (要求分隔符在前缀后)
- 团队目录不存在时返回 true (安全)

## 安全特性

1. **路径遍历防护**:
   - `..` 段过滤
   - 根/近根路径拒绝
   - 前缀攻击保护

2. **编码攻击防护**:
   - URL 编码遍历检测
   - Unicode 规范化攻击检测
   - 空字节检测

3. **符号链接安全**:
   - 真实路径解析
   - 悬挂符号链接检测
   - 符号链接循环检测

4. **平台安全**:
   - Windows 反斜杠拒绝
   - 绝对路径拒绝
   - UNC 路径拒绝

## 设计要点

1. **双层验证**:
   - 第一遍: 字符串级包含检查
   - 第二遍: 符号链接解析和真实路径比较

2. **防御深度**:
   - 多层安全检查
   - 早期拒绝明显攻击
   - 深度验证符号链接

3. **错误区分**:
   - 不同类型的 PathTraversalError
   - 有助于调试和安全审计

4. **幂等处理**:
   - 团队目录不存在时安全跳过符号链接检查
   - 无符号链接则无前缀攻击风险

## 与其他文件的关系

- **./paths.ts**: isAutoMemoryEnabled, getAutoMemPath
- **../services/analytics/growthbook.ts**: 特性门控
- **../utils/errors.ts**: getErrnoCode

## 注意事项

1. 团队内存是自动内存的子目录
2. 路径验证是负载承载的安全控制
3. 符号链接检查可能抛出各种错误
4. 悬挂符号链接是潜在的攻击向量
5. Unicode 全角字符可能绕过简单过滤
