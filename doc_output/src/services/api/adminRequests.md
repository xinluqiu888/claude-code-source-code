# adminRequests.ts — 管理员请求服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/api/adminRequests.ts`
- **所属模块**: API Service
- **功能类型**: 管理员请求管理

## 功能概述

该模块为 Team/Enterprise 用户提供管理员请求功能。没有计费/管理员权限的用户可以创建请求（配额增加或席位升级），由其组织管理员处理。

## 核心内容详解

### 类型定义

#### `AdminRequestType`
请求类型：
- `'limit_increase'` — 配额增加请求
- `'seat_upgrade'` — 席位升级请求

#### `AdminRequestStatus`
请求状态：
- `'pending'` — 待处理
- `'approved'` — 已批准
- `'dismissed'` — 已驳回

#### `AdminRequest`
完整的请求对象，包含：
- `uuid` — 请求唯一标识
- `status` — 当前状态
- `requester_uuid` — 请求者 UUID
- `created_at` — 创建时间
- `request_type` 和 `details` — 类型特定数据

### 主要函数

#### `createAdminRequest(params): Promise<AdminRequest>`
创建管理员请求。

**端点：**
```
POST /api/oauth/organizations/{orgUUID}/admin_requests
```

**行为：**
- 如果同类型的待处理请求已存在，返回现有请求
- 支持 limit_increase 和 seat_upgrade 两种类型

#### `getMyAdminRequests(requestType, statuses): Promise<AdminRequest[] | null>`
获取当前用户的特定类型待处理请求。

**端点：**
```
GET /api/oauth/organizations/{orgUUID}/admin_requests/me?request_type={type}&statuses={status}
```

#### `checkAdminRequestEligibility(requestType): Promise<AdminRequestEligibilityResponse | null>`
检查特定请求类型对该组织是否允许。

**端点：**
```
GET /api/oauth/organizations/{orgUUID}/admin_requests/eligibility?request_type={type}
```

### 认证头

所有请求使用 OAuth 认证：
- `Authorization: Bearer {accessToken}`
- `x-organization-uuid: {orgUUID}`

## 设计要点

1. **组织级操作** — 所有请求在组织级别执行
2. **防重复** — 同类型待处理请求不会重复创建
3. **类型安全** — 完整的 TypeScript 类型定义
4. **统一认证** — 使用 OAuth 头标准模式

## 与其他文件的关系

- **被调用**: UI 组件和管理员请求流程
- **依赖**: `../../utils/teleport/api.ts` 的 `getOAuthHeaders` 和 `prepareApiRequest`
- **关联**: `../../constants/oauth.ts` 配置

## 注意事项

- 仅 Team/Enterprise 组织支持管理员请求
- 需要有效的 OAuth 访问令牌和组织 UUID
- 使用 `prepareApiRequest` 确保认证就绪
