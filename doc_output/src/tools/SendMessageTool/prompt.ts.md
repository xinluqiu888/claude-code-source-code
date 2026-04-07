# prompt.ts — SendMessage工具提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/SendMessageTool/prompt.ts`
- **作用**: 构建SendMessage工具的动态提示文本

## 核心内容详解

### 描述常量

```typescript
export const DESCRIPTION = 'Send a message to another agent'
```

### 动态提示构建

```typescript
export function getPrompt(): string
```

根据`UDS_INBOX`特性开关构建不同的提示内容。

### 基础提示内容

1. **发送格式**:
   ```json
   {"to": "researcher", "summary": "assign task 1", "message": "start on task #1"}
   ```

2. **目标类型**:
   - `"researcher"`: 通过名称指定队友
   - `"*"`: 广播给所有队友（昂贵，仅必要时使用）

3. **重要说明**:
   - 纯文本输出对其他代理不可见
   - 消息自动传递，无需检查收件箱
   - 引用队友时使用名称而非UUID
   - 转发时不引用原文（已渲染给用户）

### UDS_INBOX特性内容

当`UDS_INBOX`特性启用时，额外支持：

1. **额外目标类型**:
   - `"uds:/path/to.sock"`: 本地Claude会话socket
   - `"bridge:session_..."`: 远程控制会话

2. **跨会话通信**:
   - 使用`ListPeers`发现目标
   - 消息到达时包装为`<cross-session-message from="...">`
   - 回复时复制`from`属性作为`to`

3. **协议响应（遗留）**:
   - `type: "shutdown_request"` → 响应`type: "shutdown_response"`
   - `type: "plan_approval_request"` → 响应`type: "plan_approval_response"`
   - 需要echo `request_id`和设置`approve` true/false

## 设计要点

1. **条件内容**: 根据UDS_INBOX特性开关动态构建
2. **表格格式**: 使用Markdown表格清晰说明目标类型
3. **代码示例**: 提供具体的JSON使用示例
4. **重要提示**: 明确说明阻塞要求和转发规则

## 与其他文件的关系

- **SendMessageTool.ts**: 导入DESCRIPTION和getPrompt
- **bun:bundle**: 使用`feature('UDS_INBOX')`检查特性开关

## 注意事项

- 跨会话通信需要先调用ListPeers发现目标
- 结构化消息回复必须echo原消息的request_id
- 批准shutdown将终止进程
