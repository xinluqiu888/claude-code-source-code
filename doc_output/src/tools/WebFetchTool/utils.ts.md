# utils.ts — 网页获取工具工具函数

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebFetchTool/utils.ts`
- **类型**: TypeScript 工具模块
- **功能**: 提供 WebFetch 工具的内容获取和处理函数

## 核心内容详解

### 导出函数

```typescript
export function clearWebFetchCache(): void
export function isPreapprovedUrl(url: string): boolean
export function validateURL(url: string): boolean
export function checkDomainBlocklist(domain: string): Promise<DomainCheckResult>
export function isPermittedRedirect(originalUrl: string, redirectUrl: string): boolean
export function getWithPermittedRedirects(
  url: string,
  signal: AbortSignal,
  redirectChecker: (original: string, redirect: string) => boolean,
  depth?: number
): Promise<AxiosResponse<ArrayBuffer> | RedirectInfo>
export function getURLMarkdownContent(
  url: string,
  abortController: AbortController
): Promise<FetchedContent | RedirectInfo>
export function applyPromptToMarkdown(
  prompt: string,
  markdownContent: string,
  signal: AbortSignal,
  isNonInteractiveSession: boolean,
  isPreapprovedDomain: boolean
): Promise<string>
```

### 错误类

| 类名 | 说明 |
|------|------|
| `DomainBlockedError` | 域名被阻止 |
| `DomainCheckFailedError` | 域名检查失败 |
| `EgressBlockedError` | 出口代理阻止 |

### 缓存配置

```typescript
// URL 内容缓存
const URL_CACHE = new LRUCache<string, CacheEntry>({
  maxSize: 50 * 1024 * 1024,  // 50MB
  ttl: 15 * 60 * 1000,        // 15 分钟
})

// 域名检查缓存
const DOMAIN_CHECK_CACHE = new LRUCache<string, true>({
  max: 128,
  ttl: 5 * 60 * 1000,  // 5 分钟
})
```

### 获取流程

1. **URL 验证**: 检查格式和长度（最大 2000 字符）
2. **缓存检查**: 使用 LRU 缓存
3. **域名检查**: 调用 Anthropic API 检查域名安全
4. **内容获取**: 使用 axios 获取，支持重定向
5. **内容转换**: HTML 使用 Turndown 转换为 Markdown
6. **内容处理**: 使用 Haiku 模型处理

### 重定向处理

- **允许的重定向**:
  - 添加/移除 `www.`
  - 相同源，不同路径/查询参数
- **禁止的重定向**:
  - 协议变更
  - 端口变更
  - 用户名/密码

### 安全配置

- 最大 HTTP 内容长度: 10MB
- 获取超时: 60 秒
- 域名检查超时: 10 秒
- 最大重定向数: 10

## 与其他文件的关系

### 依赖
- `axios` — HTTP 客户端
- `lru-cache` — LRU 缓存实现
- `turndown` — HTML 转 Markdown
- `../../services/api/claude.js` — Haiku 模型调用
- `./preapproved.ts` — 预批准域名检查
- `./prompt.ts` — 辅助模型提示生成

### 被依赖
- `WebFetchTool.ts` — 工具主实现文件
