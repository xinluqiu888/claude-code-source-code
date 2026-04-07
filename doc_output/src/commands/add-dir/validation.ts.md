# validation.ts — add-dir 命令验证逻辑

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/add-dir/validation.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 111 行 |
| 主要职责 | 提供目录验证逻辑和友好的错误消息生成 |

## 功能概述

该文件实现了 `/add-dir` 命令的目录验证逻辑，确保用户添加的目录符合要求。它检查路径是否存在、是否为目录、以及是否与现有工作目录冲突。同时提供结构化的错误类型和对应的用户友好错误消息。

## 核心内容详解

### 类型定义

**AddDirectoryResult** 联合类型：
```typescript
export type AddDirectoryResult =
  | { resultType: 'success'; absolutePath: string }
  | { resultType: 'emptyPath' }
  | { resultType: 'pathNotFound' | 'notADirectory'; directoryPath: string; absolutePath: string }
  | { resultType: 'alreadyInWorkingDirectory'; directoryPath: string; workingDir: string }
```

### 主要函数

**validateDirectoryForWorkspace**
- 参数：`directoryPath` (路径字符串)、`permissionContext` (权限上下文)
- 返回值：`AddDirectoryResult`
- 功能：验证目录是否可以添加为工作目录

验证流程：
1. 检查空路径
2. 解析绝对路径（使用 `resolve(expandPath(directoryPath))`）
3. 使用 `stat` 系统调用检查路径是否存在且为目录
4. 处理访问权限错误（EACCES/EPERM）作为 "not found"
5. 检查是否已在现有工作目录中

**addDirHelpMessage**
- 参数：`AddDirectoryResult`
- 返回值：用户友好的错误消息字符串
- 使用 `chalk` 添加颜色高亮

### 错误处理策略

对于 `stat` 调用失败的情况，特别处理以下错误码：
- `ENOENT`: 路径不存在
- `ENOTDIR`: 路径不是目录
- `EACCES`/`EPERM`: 权限不足（避免启动时因配置目录不可访问而崩溃）

## 设计要点

1. **结构化返回**：使用联合类型而非抛出异常，调用方可根据 `resultType` 进行精确处理
2. **路径规范化**：使用 `resolve()` 去除尾部斜杠，确保 `/foo` 和 `/foo/` 映射到相同存储键
3. **防御性编程**：将权限错误视为"未找到"而非崩溃，提升健壮性
4. **用户体验**：错误消息提供上下文和建议（如建议添加父目录）

## 与其他文件的关系

- **add-dir.tsx**: 调用验证函数并根据结果展示相应UI
- **errors.ts**: 使用 `getErrnoCode` 提取错误码
- **path.ts**: 使用 `expandPath` 展开路径（如 `~` 展开）
- **permissions/filesystem.ts**: 使用 `allWorkingDirectories` 和 `pathInWorkingPath` 检查目录冲突

## 注意事项

1. **路径一致性**：`resolve()` 会去除尾部斜杠，这是有意设计以避免重复存储
2. **权限错误处理**：特意将 EACCES/EPERM 视为 "not found" 以防止启动崩溃
3. **异步验证**：整个验证流程是异步的，支持非阻塞 I/O
4. **单系统调用**：通过 `stat` 一次性检查存在性和目录类型，优化性能
