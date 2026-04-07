# session/session.tsx

## 文件描述
Session 组件 - 远程会话URL和二维码展示

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 显示远程会话连接信息 |
| 组件 | QRCode, Text |

## 函数概述

### call
主组件函数，渲染会话连接信息：
1. 生成会话URL
2. 生成二维码
3. 显示连接信息

### 函数签名
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
): Promise<React.ReactNode | null>
```

## 核心内容

### 组件渲染
```typescript
return (
  <Box flexDirection="column">
    <Text>Scan the QR code to connect to this session:</Text>
    <QRCode value={sessionUrl} />
    <Text>Or visit: {sessionUrl}</Text>
  </Box>
);
```

### URL生成
- 基于会话ID生成唯一URL
- 包含必要的认证信息
- 支持自定义域名

### 二维码显示
- 使用 QRCode 组件渲染
- 适配终端显示

## 设计点

1. **移动友好**：二维码便于手机扫描
2. **多方式连接**：提供扫码和直接访问两种方式
3. **安全考虑**：URL包含会话密钥
4. **简洁界面**：清晰显示连接信息

## 与其他文件的关系

- 使用 `Box` 和 `Text` 组件进行布局
- 导入 `QRCode` 组件显示二维码
- 从 context 获取会话信息

## 注意事项

- URL 需要包含有效的会话密钥
- 二维码大小需适配终端
- 会话URL有过期时间
- 需要处理无远程会话的情况
