# preapproved.ts — 预批准域名列表

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebFetchTool/preapproved.ts`
- **类型**: TypeScript 常量模块
- **功能**: 定义 WebFetch 工具可以安全访问的预批准域名列表

## 核心内容详解

### 安全警告

```typescript
// SECURITY WARNING: These preapproved domains are ONLY for WebFetch (GET requests only).
// The sandbox system deliberately does NOT inherit this list for network restrictions.
```

预批准域名仅用于 WebFetch（仅 GET 请求）。沙盒系统不会继承此列表用于网络限制。

### 导出常量

```typescript
export const PREAPPROVED_HOSTS = new Set([...])

export function isPreapprovedHost(hostname: string, pathname: string): boolean
```

### 预批准域名分类

#### Anthropic 相关
- `platform.claude.com`
- `code.claude.com`
- `modelcontextprotocol.io`
- `github.com/anthropics`
- `agentskills.io`

#### 编程语言文档
- Python, C/C++, Java, C#/.NET, JavaScript, Go, PHP, Swift, Kotlin, Ruby, Rust, TypeScript

#### Web 和 JavaScript 框架
- React, Angular, Vue.js, Next.js, Express.js, Node.js, Bun, jQuery, Bootstrap, Tailwind CSS, D3.js, Three.js, Redux, Webpack, Jest, React Router

#### Python 框架和库
- Django, Flask, FastAPI, Pandas, NumPy, TensorFlow, PyTorch, Scikit-learn, Matplotlib, Requests, Jupyter

#### 其他框架
- PHP (Laravel, Symfony, WordPress)
- Java (Spring, Hibernate, Tomcat, Gradle, Maven)
- .NET (ASP.NET, NuGet, Blazor)
- 移动开发 (React Native, Flutter, iOS, Android)
- 数据科学 (Keras, Apache Spark, Hugging Face, Kaggle)
- 数据库 (MongoDB, Redis, PostgreSQL, MySQL, SQLite, GraphQL, Prisma)
- 云和 DevOps (AWS, Google Cloud, Azure, Kubernetes, Docker, Terraform, Ansible, Vercel, Netlify, Heroku)
- 测试 (Cypress, Selenium)
- 游戏开发 (Unity, Unreal Engine)
- 其他工具 (Git, Nginx, Apache HTTP Server)

### isPreapprovedHost 函数

检查域名和路径是否在预批准列表中：

1. 精确主机名匹配（O(1) Set.has）
2. 路径前缀匹配（支持如 `github.com/anthropics`）
3. 强制执行路径段边界（防止 `/anthropics-evil/malware` 匹配 `/anthropics`）

## 与其他文件的关系

### 被依赖
- `WebFetchTool.ts` — 权限检查
- `utils.ts` — URL 预批准检查
