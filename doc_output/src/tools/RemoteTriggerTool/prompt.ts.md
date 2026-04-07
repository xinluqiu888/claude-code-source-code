# prompt.ts — RemoteTrigger工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/RemoteTriggerTool/prompt.ts`
- **作用**: 定义RemoteTrigger工具的名称、描述和提示文本

## 核心内容详解

### 工具名称常量

```typescript
export const REMOTE_TRIGGER_TOOL_NAME = 'RemoteTrigger'
```

### 描述常量

```typescript
export const DESCRIPTION =
  'Manage scheduled remote Claude Code agents (triggers) via the claude.ai CCR API. Auth is handled in-process — the token never reaches the shell.'
```

说明该工具通过claude.ai CCR API管理远程Claude Code代理（触发器），身份验证在进程内处理，token不会暴露到shell。

### 提示文本常量

```typescript
export const PROMPT = `Call the claude.ai remote-trigger API. Use this instead of curl — the OAuth token is added automatically in-process and never exposed.

Actions:
- list: GET /v1/code/triggers
- get: GET /v1/code/triggers/{trigger_id}
- create: POST /v1/code/triggers (requires body)
- update: POST /v1/code/triggers/{trigger_id} (requires body, partial update)
- run: POST /v1/code/triggers/{trigger_id}/run

The response is the raw JSON from the API.`
```

支持的5种操作：
1. **list**: 列出所有触发器
2. **get**: 获取特定触发器详情
3. **create**: 创建新触发器（需要body）
4. **update**: 更新触发器（需要body，部分更新）
5. **run**: 运行触发器

## 设计要点

1. **安全认证**: OAuth token自动在进程内添加，不暴露到shell
2. **替代curl**: 封装了claude.ai remote-trigger API调用
3. **原始JSON响应**: 直接返回API的原始JSON响应

## 与其他文件的关系

- **RemoteTriggerTool.ts**: 导入使用这些常量
