# Token 认证体系（基于 FastAPI 实现）

## 一、为什么采用 Token 认证

Token JSON 格式（OAuth2.1/OIDC 兼容结构）因其标准化、可扩展、安全且易于集成的特点 已被广泛应用于多种现代 Web 和移动应用场景。

以下是其主要适用场景及原因分析：

### 1. 单页应用（SPA）与前后端分离架构

- 典型技术栈：React、Vue、Angular + Node.js/Go/Python 后端
- 适用原因：
    - 前端通过 fetch/axios 获取 access_token，用于调用 API。
    - 使用 expires_in 或 expires_at 在内存中管理 token 生命周期。
    - 支持自动刷新机制（401 拦截 → 调用 /refresh → 重试请求）。
- 示例：
    - 用户登录后，前端存储 token 并用于请求用户资料、订单等资源。

### 2. 移动应用（iOS/Android）

- 典型场景：原生 App 调用后端 Restful 或 GraphQL API
- 适用原因：
    - 标准 JSON 格式便于移动端解析（Swift/Kotlin/Flutter）。
    - refresh_token 机制减少频繁登录，提升用户体验。
    - scope 字段支持细粒度权限控制（如“仅访问相册”）。
- 安全建议：
    - refresh_token 存储在安全密钥链（Keychain/Keystore）中。

### 3. 第三方登录与 OpenID Connect（OIDC）集成

- 典型服务：Google、GitHub、Auth0、Azure AD、Okta、Firebase Auth
- 适用原因：
    - 现代格式直接支持 id_token（JWT），包含用户身份信息（如 sub, email）。
    - user 字段可内联基本信息，减少额外查询。
    - 完全兼容 OpenID Connect Core 1.0。
- 示例：
    - 用户点击“Login with Google”，后端返回标准 token + 用户信息。

### 4. 微服务架构与 API 网关

- 典型场景：多个服务间通过 JWT 进行身份传递
- 适用原因：
    - access_token 为 JWT，可自包含用户信息和权限（scope/role）。
    - 服务间无需查询数据库即可完成鉴权（无状态）。
    - 格式统一，便于网关统一处理认证逻辑。
- 示例： API 网关验证 token 后，将用户信息注入 header 转发给下游服务。

### 5. Serverless 与 Jamstack 应用

- 典型平台：Vercel、Netlify、AWS Lambda、Cloudflare Workers
- 适用原因：
    - 无状态特性匹配 Serverless 架构。
    - 前端静态部署，通过 /api/auth/refresh 获取 token。
    - 支持边缘函数（Edge Functions）快速验证 JWT。
- 示例：Next.js App Router 中使用 next-auth，返回标准 token 格式。

### 6. 多端统一认证（Web + App + 小程序）

- 典型需求：同一套后端支持 Web、iOS、Android、微信小程序
- 适用原因：
    - 标准 JSON 格式跨平台兼容。
    - 所有客户端可统一处理 access_token 和刷新逻辑。
    - 减少后端为不同客户端定制接口的成本。
- 示例：一个电商平台，Web 和 App 共用 /auth/token 接口。

### 7. 实时应用（WebSocket / SSE）

- 典型场景：聊天、通知、协同编辑
- 适用原因：
    - 连接建立时通过 URL 或 header 传递 access_token。
    - 服务端验证 token 后建立用户会话。
    - sub 或 user.id 可用于标识连接用户。
- 注意：长连接期间 token 过期需设计重连机制。

### 8. B2B / SaaS 多租户系统

- 典型场景：企业级应用，支持多个组织/租户
- 适用原因：
    - scope 可定义租户内权限（如 org:read, billing:write）。
    - user 字段可包含 tenant_id、role 等上下文信息。
    - 刷新机制支持长期会话（如管理员后台）。
- 示例：CRM 系统中，token 携带 tenant_id 和 role: sales_manager。

## 二、 常见的 Token 格式

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "JsZl9zaGFvI1NiIsInVuYW50Ijp0cnVlLCJ0ZW5...",
  "refresh_expires_in": 1209600,
  "scope": "read write profile",
  "id_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "sub": "usr_123",
    "...": "......"
  },
  "error": "invalid_grant",
  "error_description": "Refresh token has expired",
  "error_code": "token_expired",
  "timestamp": "2025-10-12T16:10:00Z"
}
```

### 1. 字段说明

| 字段                   | 类型     | 来源             | 说明                                                             |
|----------------------|--------|----------------|----------------------------------------------------------------|
| `access_token`       | string | JWT            | 用于访问受保护资源的令牌，需在请求头 `Authorization: Bearer <token>` 中携带         |
| `token_type`         | string | -              | 令牌类型，固定为 `Bearer`，符合 RFC 6750 标准                               |
| `expires_in`         | number | -              | 表示 `access_token` 的有效期（单位：秒），便于客户端计算过期时间                       |
| `refresh_token`      | string | -              | 用于获取新的 `access_token`，应安全存储，避免泄露                               |
| `refresh_expires_in` | number | -              | 表示 `refresh_token` 的有效期（单位：秒），用于管理长期会话                         |
| `scope`              | string | OAuth2         | 当前令牌被授予的权限范围，以空格分隔（如 `read write profile`）                     |
| `id_token`           | string | OpenID Connect | ID Token（JWT 格式），包含用户身份信息，用于单点登录（SSO）场景                        |
| `user`               | object | 自定义扩展          | 内嵌用户基本信息（如 `sub`, `email`, `name` 等），减少首次加载时额外请求               |
| `error`              | string | OAuth2         | 标准化的错误码（如 `invalid_grant`, `invalid_request`），用于程序判断错误类型       |
| `error_description`  | string | -              | 可读的错误描述，用于调试和用户提示                                              |
| `error_code`         | string | 自定义扩展          | 业务层面的错误代码（如 `token_expired`, `invalid_credentials`），便于前端处理异常流程 |
| `timestamp`          | string | ISO 8601       | 错误发生的时间戳（UTC），格式为 `YYYY-MM-DDTHH:MM:SSZ`，用于日志追踪与审计             |

### 2. user 字段说明

```
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "...": "......",
  "user": {
    "sub": "usr_123",
    "name": "Alice Chen",
    "email": "alice@example.com",
    "email_verified": true,
    "picture": "https://avatar.example.com/a123.jpg",
    "tenant_id": "tenant_456"
  },
  "...": "......"
}
```

| 字段               | 类型           | 来源 / 标准 | 说明                                                         |
|------------------|--------------|---------|------------------------------------------------------------|
| `sub`            | string       | OIDC    | **Subject Identifier**，用户唯一标识符（在同一 issuer 下唯一），推荐作为系统内用户主键 |
| `name`           | string       | OIDC    | 用户全名（如 "Alice Chen"），通常用于界面显示                              |
| `email`          | string       | OIDC    | 用户邮箱地址，常用于登录和通知                                            |
| `email_verified` | boolean      | OIDC    | 邮箱是否已验证，`true` 表示通过验证                                      |
| `picture`        | string (URL) | OIDC    | 用户头像图片的 URL 地址                                             |
| `tenant_id`      | string       | 自定义扩展   | 多租户系统中，用户所属租户的唯一标识                                         |

> 💡 **使用建议**：
> - **核心字段**（如 `sub`, `email`, `name`）建议遵循 OIDC 标准，提升兼容性。
> - **权限相关字段**（`role`, `permissions`）应结合后端鉴权系统使用。
> - **按需返回**：生产环境应根据客户端权限动态裁剪 `user` 字段，遵循最小权限原则。

## 三、 API 接口说明

### 1. /auth/token 接口

请求与响应格式说明，遵循 OAuth 2.1 标准

✅ 请求头

- method: POST
- Content-Type: application/x-www-form-urlencoded

✅ 请求参数

| 参数名      | 类型     | 是否必需            | 说明           |
|----------|--------|-----------------|--------------|
| username | string | 仅 password 模式必需 | 用户名（如邮箱或登录名） |
| password | string | 仅 password 模式必需 | 用户密码         |

✅ 示例请求

```
POST /auth/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

username=admin&password=secret123
```

✅ 响应格式

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900,
  "refresh_token": "JsZl9zaGFvI1NiIsInVuYW50Ijp0cnVlLCJ0ZW5...",
  "refresh_expires_in": 1209600,
  "scope": "",
  "user": {
    "sub": null,
    "name": "管理员",
    "email": "admin@example.com",
    "picture": "https://avatar.example.com/a123.jpg",
    "tenant_id": "a1b2c3d45e6f7g8h9i"
  }
}
```

| 字段                 | 类型     | 来源 / 标准   | 说明                                    |
|--------------------|--------|-----------|---------------------------------------|
| access_token       | string | OAuth 2.1 | JWT 格式的访问令牌，用于后续请求                    |
| token_type         | string | OAuth 2.1 | 固定为 Bearer，表示使用 Bearer Token 认证       |
| expires_in         | number | OAuth 2.1 | access_token 有效期（秒），如 900 = 15 分钟     |
| refresh_token      | string | OAuth 2.1 | 用于获取新 access_token 的刷新令牌              |
| refresh_expires_in | number | 扩展        | refresh_token 有效期（秒），如 1209600 = 14 天 |
| scope              | string | OAuth 2.1 | 当前令牌被授予的权限范围                          |
| user               | object | 扩展        | 内嵌用户信息，避免首次请求额外调用 /auth/me            |

user 字段 可支持的清单

| 字段             | 类型            | 来源 / 标准 | 已支持 | 说明                                             |
|----------------|---------------|---------|-----|------------------------------------------------|
| sub            | string        | OIDC    | 空值  | ，待后续看是否需要提供 **Subject Identifier**             |
| name           | string        | OIDC    | 是   | 用户全名（如 "Alice Chen"），通常用于界面显示                  |
| email          | string        | OIDC    | 空值  | 用户邮箱地址，常用于登录和通知                                |
| email_verified | boolean       | OIDC    | 空值  | 邮箱是否已验证，`true` 表示通过验证                          |
| picture        | string (URL)  | OIDC    | 空值  | 空值（待支持），用户头像图片的 URL 地址                         |
| role           | string        | 自定义扩展   | 不支持 | 用户角色（如 `admin`, `user`, `editor`），用于 RBAC 权限控制 |
| roles          | array<string> | 自定义扩展   | 不支持 | 用户拥有的多个角色列表，支持多角色场景                            |
| groups         | array<string> | 自定义扩展   | 不支持 | 用户所属用户组或部门（如 `marketing`, `engineering`）       |
| organization   | string        | 自定义扩展   | 不支持 | 用户所属组织名称                                       |
| department     | string        | 自定义扩展   | 不支持 | 用户所在部门                                         |
| job_title      | string        | 自定义扩展   | 不支持 | 职位/职称（如 "Senior Engineer"）                     |
| tenant_id      | string        | 自定义扩展   | 不支持 | 多租户系统中，用户所属租户的唯一标识                             |

- 刷新 Token（grant_type=refresh_token）

```
POST /auth/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.xxxxx
```