# secretScanner.ts — 密钥扫描器

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/teamMemorySync/secretScanner.ts`
- **作用域**: 团队记忆的客户端密钥扫描（PSR M22174）
- **主要导出**:
  - `scanForSecrets`: 扫描内容中的密钥
  - `redactSecrets`: 脱敏内容中的密钥
  - `getSecretLabel`: 获取规则 ID 的标签

## 功能概述

在内容上传前扫描凭证，确保密钥不会离开用户机器。使用 gitleaks 的高置信度规则子集（https://github.com/gitleaks/gitleaks，MIT 许可证）。

## 核心内容详解

### 规则定义

```typescript
type SecretRule = {
  id: string       // Gitleaks 规则 ID（kebab-case）
  source: string   // 正则源码，首次扫描时延迟编译
  flags?: string   // 可选的 JS 正则标志
}

type SecretMatch = {
  ruleId: string   // 匹配的规则 ID
  label: string    // 人类可读的标签
}
```

### 扫描规则

包含 30+ 个高置信度规则，按类别组织：

#### 云提供商
- `aws-access-token`: AWS 访问令牌
- `gcp-api-key`: GCP API 密钥
- `azure-ad-client-secret`: Azure AD 客户端密钥
- `digitalocean-pat`: DigitalOcean PAT

#### AI APIs
- `anthropic-api-key`: Anthropic API 密钥（运行时组装前缀）
- `anthropic-admin-api-key`: Anthropic 管理员密钥
- `openai-api-key`: OpenAI API 密钥
- `huggingface-access-token`: HuggingFace 访问令牌

#### 版本控制
- `github-pat`: GitHub PAT
- `github-fine-grained-pat`: GitHub 细粒度 PAT
- `github-oauth`: GitHub OAuth
- `gitlab-pat`: GitLab PAT

#### 通信
- `slack-bot-token`: Slack Bot 令牌
- `slack-user-token`: Slack 用户令牌
- `twilio-api-key`: Twilio API 密钥
- `sendgrid-api-token`: SendGrid API 令牌

#### 开发工具
- `npm-access-token`: NPM 访问令牌
- `pypi-upload-token`: PyPI 上传令牌
- `databricks-api-token`: Databricks API 令牌
- `hashicorp-tf-api-token`: HashiCorp TF API 令牌

#### 可观测性
- `grafana-api-key`: Grafana API 密钥
- `grafana-cloud-api-token`: Grafana Cloud 令牌
- `sentry-user-token`: Sentry 用户令牌

#### 支付/商务
- `stripe-access-token`: Stripe 访问令牌
- `shopify-access-token`: Shopify 访问令牌

#### 加密
- `private-key`: 私钥（各种格式）

### 核心函数

#### `scanForSecrets(content)`
扫描字符串中的潜在密钥：
1. 延迟编译规则（首次扫描）
2. 测试每个规则
3. 去重（按规则 ID）
4. 返回匹配列表（不包含实际匹配文本）

**注意**: 故意不返回实际匹配文本，避免日志或显示中泄露密钥。

#### `redactSecrets(content)`
将匹配的内容脱敏为 `[REDACTED]`：
- 使用全局标志重新编译规则
- 仅替换捕获组（保留边界字符）
- 返回脱敏后的内容

#### `getSecretLabel(ruleId)` / `ruleIdToLabel(ruleId)`
将 kebab-case 规则 ID 转换为人类可读标签：
```typescript
"github-pat" → "GitHub PAT"
"aws-access-token" → "AWS Access Token"
```

特殊大小写映射：
- aws, gcp, api, pat, oauth, npm, pypi, jwt 等
- GitHub, GitLab, OpenAI, DigitalOcean, HuggingFace, HashiCorp, SendGrid

### 延迟编译

```typescript
let compiledRules: Array<{ id: string; re: RegExp }> | null = null
let redactRules: RegExp[] | null = null

function getCompiledRules(): Array<{ id: string; re: RegExp }> {
  if (compiledRules === null) {
    compiledRules = SECRET_RULES.map(r => ({
      id: r.id,
      re: new RegExp(r.source, r.flags),
    }))
  }
  return compiledRules
}
```

### Anthropic 密钥前缀

运行时组装避免字面量出现在外部包中：
```typescript
const ANT_KEY_PFX = ['sk', 'ant', 'api'].join('-')  // "sk-ant-api"
```

## 设计要点

1. **高置信度**: 仅使用具有独特前缀的规则
2. **客户端扫描**: 密钥在上传前被拦截
3. **隐私保护**: 不返回或记录实际密钥值
4. **延迟编译**: 正则首次使用时编译
5. **运行时组装**: 敏感前缀运行时组装
6. **全局脱敏**: `redactSecrets` 替换所有匹配

## 与其他文件的关系

- **index.ts**: `readLocalTeamMemory` 调用 `scanForSecrets`
- **teamMemSecretGuard.ts**: 编辑工具调用 `scanForSecrets`

## 注意事项

1. **JS 正则差异**: Go 正则的 `(?i)` 和 `(?-i:...)` 在 JS 中重写为字符类
2. **边界字符**: 模式包含边界字符（空格、引号、分号）
3. **规则排序**: 按在开发团队内容中出现的可能性排序
4. **MIT 许可证**: 规则源自 gitleaks（MIT 许可证）
