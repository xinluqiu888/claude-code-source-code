# filesApi.ts — 文件 API 客户端

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/filesApi.ts`
- **所属模块**: API Service
- **功能类型**: 文件上传下载管理

## 功能概述

该模块提供与 Anthropic Public Files API 的交互功能，支持文件下载（BYOC 模式）和上传。用于 Claude Code Agent 在会话启动时下载文件附件。

## 核心内容详解

### 常量配置

```typescript
FILES_API_BETA_HEADER = 'files-api-2025-04-14,oauth-2025-04-20'
ANTHROPIC_VERSION = '2023-06-01'
MAX_RETRIES = 3
BASE_DELAY_MS = 500
MAX_FILE_SIZE_BYTES = 500 * 1024 * 1024  // 500MB
DEFAULT_CONCURRENCY = 5
```

### 类型定义

#### `File`
文件规范：
```typescript
{
  fileId: string      // 文件 ID（如：file_011CNha8iCJcU1wXNR6q4V8w）
  relativePath: string // 相对路径
}
```

#### `FilesApiConfig`
API 配置：
```typescript
{
  oauthToken: string   // OAuth 令牌（来自会话 JWT）
  baseUrl?: string     // API 基础 URL
  sessionId: string    // 会话 ID（用于目录创建）
}
```

#### `DownloadResult`
下载结果：
```typescript
{
  fileId: string
  path: string
  success: boolean
  error?: string
  bytesWritten?: number
}
```

### 下载功能

#### `downloadFile(fileId, config): Promise<Buffer>`
下载单个文件。

**端点：**
```
GET /v1/files/{fileId}/content
```

**重试逻辑：**
- 指数退避重试（最多 3 次）
- 非 500 错误直接抛出
- 404/401/403 为非重试错误

#### `buildDownloadPath(basePath, sessionId, relativePath): string | null`
构建下载路径。

**安全性：**
- 规范化路径（`path.normalize`）
- 防止目录遍历（检查 `..` 前缀）
- 移除冗余前缀

**输出格式：**
```
{basePath}/{sessionId}/uploads/{relativePath}
```

#### `downloadAndSaveFile(attachment, config): Promise<DownloadResult>`
下载并保存文件。

**流程：**
1. 构建完整路径
2. 下载文件内容
3. 创建父目录
4. 写入文件
5. 返回结果

#### `downloadSessionFiles(files, config, concurrency): Promise<DownloadResult[]>`
并行下载多个文件。

**特性：**
- 限制并发数（默认 5）
- 保持输入顺序输出
- 记录成功数和耗时

### 上传功能（BYOC 模式）

#### `uploadFile(filePath, relativePath, config): Promise<UploadResult>`
上传单个文件。

**端点：**
```
POST /v1/files
```

**流程：**
1. 读取文件内容
2. 大小验证（500MB 限制）
3. 构建 multipart/form-data 请求体
4. 指数退避重试上传

**请求体构造：**
- 文件部分：`Content-Type: application/octet-stream`
- Purpose：`user_data`
- Boundary：使用 `crypto.randomUUID` 避免冲突

#### `uploadSessionFiles(files, config, concurrency): Promise<UploadResult[]>`
并行上传多个文件。

### 文件列表功能

#### `listFilesCreatedAfter(afterCreatedAt, config): Promise<FileMetadata[]>`
列出指定时间后创建的文件。

**端点：**
```
GET /v1/files?after_created_at={timestamp}&after_id={cursor}
```

**分页：**
- 服务器默认每页 500，最大 1000
- 使用 `after_id` 游标分页
- 自动处理 `has_more`

### 工具函数

#### `parseFileSpecs(fileSpecs): File[]`
解析 CLI 文件规范。

**格式：**
```
<file_id>:<relative_path>
```

**支持：**
- 多个规范以空格分隔
- 自动展开嵌套空格

## 设计要点

1. **安全路径处理** — 防止目录遍历攻击
2. **并发控制** — 限制并行操作避免资源耗尽
3. **指数退避** — 网络错误时智能重试
4. **非重试错误** — 401/403/404/413 立即失败
5. **多部分编码** — 手动构建 multipart 请求
6. **分页支持** — 自动处理大文件列表

## 与其他文件的关系

- **被调用**: Agent 会话启动流程
- **依赖**: `../../utils/sessionIngressAuth.ts` 获取 JWT 令牌
- **关联**: 分析事件 `tengu_file_upload_failed` / `tengu_file_list_failed`

## 注意事项

- OAuth 令牌来自会话 JWT，非标准 OAuth 流程
- 上传文件大小限制 500MB
- 下载超时 60 秒，上传超时 2 分钟
- 文件保存到 `{cwd}/{sessionId}/uploads/` 目录
