# prompt.ts — 网页获取工具提示信息

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebFetchTool/prompt.ts`
- **类型**: TypeScript 提示信息模块
- **功能**: 定义 WebFetch 工具的提示信息和辅助函数

## 核心内容详解

### 导出内容

```typescript
export const WEB_FETCH_TOOL_NAME = 'WebFetch'

export const DESCRIPTION = `...`

export function makeSecondaryModelPrompt(
  markdownContent: string,
  prompt: string,
  isPreapprovedDomain: boolean
): string
```

### DESCRIPTION 常量

包含：
- 工具用途说明
- 输入参数说明（URL 和提示）
- 输出说明（模型响应）
- 使用注意事项：
  - 优先使用 MCP 提供的网页获取工具
  - URL 必须是完整有效格式
  - HTTP 自动升级到 HTTPS
  - 只读工具，不修改文件
  - 15 分钟缓存
  - 重定向处理说明
  - GitHub URL 建议使用 gh CLI

### makeSecondaryModelPrompt 函数

为 Haiku 模型生成提示：

#### 预批准域名指南
```
Provide a concise response based on the content above.
Include relevant details, code examples, and documentation excerpts as needed.
```

#### 普通域名指南
```
Provide a concise response based only on the content above.
- 125 字符最大引用限制
- 使用引号标记原文
- 外部引用语言不应逐字相同
- 不评论提示合法性
- 不产生精确歌词
```

## 与其他文件的关系

### 被依赖
- `WebFetchTool.ts` — 工具主实现文件
- `utils.ts` — 内容处理工具函数
