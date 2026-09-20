```yaml
title: Web API
language_tabs:
  - shell: Shell
  - http: HTTP
  - javascript: JavaScript
  - ruby: Ruby
  - python: Python
  - php: PHP
  - java: Java
  - go: Go
toc_footers: []
includes: []
search: true
code_clipboard: true
highlight_theme: darkula
headingLevel: 2
generator: "@tarslib/widdershins v4.0.30"
```

# API 设计规范

本文档约定本项目 Web API 的设计规范，新增接口需遵循此规范。

## 1. URL 设计

使用名词复数表示资源，避免动词。

- ✅ `/users` `/orders`
- ❌ `/getUser` `/createOrder`

层级关系用嵌套 URL 表达，但嵌套不宜过深（最多两层）。

- ✅ `/users/{userId}/orders`
- ❌ `/users/{userId}/orders/{orderId}/items/{itemId}/details`

资源名使用小写字母和连字符（kebab-case），避免下划线或驼峰。

- ✅ `/user-profiles`
- ❌ `/user_profiles` 或 `/userProfiles`

避免在 URL 中使用文件扩展名（如 `.json`、`.xml`）。

## 2. HTTP 方法

| 方法  | 用途  | 幂等性 | 安全性 |
| --- | --- | --- | --- |
| GET | 查询资源 | 是   | 是   |
| POST | 创建资源或触发操作 | 否   | 否   |
| PUT | 全量更新资源 | 是   | 否   |
| PATCH | 部分更新资源 | 否   | 否   |
| DELETE | 删除资源 | 是   | 否   |

- GET 请求不应改变资源状态。
- POST 用于创建新资源，返回 201 Created 及资源位置。
- PUT 要求客户端提供完整资源对象，用于替换。
- PATCH 仅提交需要修改的字段。
- DELETE 成功返回 204 No Content（或 200 带响应体）。

## 3. 请求与响应格式

- 统一使用 JSON 作为数据交换格式（除非明确需要其他格式，如文件上传下载）。
- 请求和响应头部应包含：
  - `Content-Type: application/json`（请求含 body 时）
  - `Accept: application/json`
- 日期时间统一使用 ISO 8601 格式，UTC 时间，如 `2025-04-01T12:30:00Z`。
- ID 类型：若无特殊需求，使用字符串（UUID）或整数，保持全局一致。
- 布尔值使用 `0`/`1` 表示（保持与本项目既有实现一致），不要使用字符串。

## 4. HTTP 状态码

| 状态码 | 含义  | 使用场景 |
| --- | --- | --- |
| 200 | OK  | GET、PUT、PATCH 成功返回数据 |
| 201 | Created | POST 成功创建资源，返回新资源或 Location |
| 204 | No Content | DELETE 成功，或无返回体的操作成功 |
| 400 | Bad Request | 请求参数错误、格式错误、校验失败 |
| 401 | Unauthorized | 未认证或认证失败 |
| 403 | Forbidden | 已认证但无权限访问 |
| 404 | Not Found | 资源不存在或无权查看 |
| 409 | Conflict | 资源状态冲突（如重复创建、版本冲突） |
| 422 | Unprocessable Entity | 语义错误，请求格式正确但业务校验失败 |
| 429 | Too Many Requests | 触发速率限制 |
| 500 | Internal Server Error | 服务器内部错误，不应返回具体错误细节 |
| 503 | Service Unavailable | 服务暂时不可用（如维护、过载） |

## 5. 错误响应格式

所有非 2xx 响应应包含统一的错误结构，便于客户端处理：

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数校验失败",
    "details": [
      {
        "field": "email",
        "issue": "格式不正确"
      }
    ],
    "traceId": "abc123def456"
  }
}
```

- `code`：机器可读的错误码（大写蛇形，如 `INVALID_PARAMETER`、`RESOURCE_NOT_FOUND`）。
- `message`：人类可读的简要描述。
- `details`：可选，具体错误细节（数组形式，可用于字段级错误）。
- `traceId`：可选，用于追踪请求日志。

## 6. JSON 字段名

- 使用 camelCase（如 `userId`、`createdAt`）。
- URL 参数：建议与 JSON 字段区分，查询参数可使用 camelCase 以保持一致（很多框架默认绑定），但需全局统一。此处推荐 camelCase 用于查询参数和 JSON 字段，保持一致性。
- 枚举值：使用大写蛇形（如 `ACTIVE`、`PENDING_REVIEW`）。

# Web API

Base URLs:

# Authentication

- HTTP Authentication, scheme: bearer

# PBC-120/Auth

## POST login

POST /auth/login

Login API, get token

> Body 请求参数

```json
{
    "username": "admin",
    "password": "admin123"
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 否   |     | none |
| » username | body | string | 是   | Username | User login username |
| » password | body | string | 是   | Password | User login password |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": {
    "roleId": 0,
    "token": "string",
    "username": "string"
  },
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | object¦null | true | none |     | none |
| »» roleId | integer | true | none |     | none |
| »» token | string | true | none |     | none |
| »» username | string | true | none |     | none |
| » message | string | true | none |     | none |

## GET check-login

GET /auth/check

Check login status, return 401 when expired

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## POST update-password

POST /auth/update-password

Update password

> Body 请求参数

```json
{
    "old_password": "admin123",
    "new_password": "admin",
    "username": "admin"
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 否   |     | none |
| » old_password | body | string | 是   |     | Admin and super admin can omit old password |
| » new_password | body | string | 是   |     | none |
| » username | body | string | 是   |     | none |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": "string",
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## GET list-user

GET /auth/list-user

List all users

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": [
    {
      "roleId": 0,
      "username": "string"
    }
  ],
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [object] | true | none |     | none |
| »» roleId | integer | true | none |     | Role id, 1 guest, 2 normal user, 4 admin, 8 super admin |
| »» username | string | true | none |     | none |
| » message | string | true | none |     | none |

## POST create-user

POST /auth/create-user

创建用户

> Body 请求参数

```json
{
    "username": "guest",
    "password": "guest",
    "roleId": 2
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 否   |     | none |
| » username | body | string | 是   |     | none |
| » password | body | string | 是   |     | none |
| » roleId | body | integer | 是   |     | 角色id，1访客，2普通用户，4管理员，8超级管理员 |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": "string",
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## DELETE delete-user

DELETE /auth/delete-user

删除用户

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| username | query | string | 是   |     | none |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": "string",
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## POST update-role

POST /auth/update-role

更新用户角色

> Body 请求参数

```json
{
    "username": "guest",
    "roleId": 4
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 否   |     | none |
| » username | body | string | 是   |     | none |
| » roleId | body | integer | 是   |     | 角色id，1访客，2普通用户，4管理员，8超级管理员 |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": "string",
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

# PBC-120/File

## POST upload-chunk

POST /file/upload-chunk

Upload file in chunks, chunks must be uploaded in order

> Body 请求参数

```yaml
fileName: ""
fileSize: ""
chunkSize: ""
currentSize: ""
index: ""
data: ""
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » fileName | body | string | 是   |     | Filename |
| » fileSize | body | string | 是   |     | File size (bytes) |
| » chunkSize | body | string | 是   |     | Chunk size (bytes) |
| » currentSize | body | string | 是   |     | Current upload chunk size (bytes) |
| » index | body | string | 是   |     | Chunk index |
| » data | body | string(binary) | 是   |     | Chunk file binary data |

> 返回示例

> 200 Response

```json
{
  "code": 406,
  "data": "failed",
  "message": "error.api.request_key_required?key=fileName"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## POST upload-validate

POST /file/upload-validate

Validate uploaded file MD5

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| fileName | query | string | 是   |     | Filename |
| fileSize | query | integer | 是   |     | File size (bytes) |
| md5 | query | string | 是   |     | File MD5 |

> 返回示例

> 200 Response

```json
{
  "code": 406,
  "data": null,
  "message": "error.api.request_key_required?key=fileName"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST remove-upload-cache

POST /file/remove-upload

Remove temporary file by filename

> Body 请求参数

```json
{
    "filename": "test.txt"
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » filename | body | string | 是   |     | Filename |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": "string",
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## GET download-largefile

GET /file/download-largefile

大文件下载，目前只用于下载sd卡录像文件和录像截图

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| filetype | query | integer | 否   |     | 文件类型 |
| filename | query | string | 否   |     | 文件名 |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| filetype | 0   |
| filetype | 1   |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

# PBC-120/Stream Media

## GET protocol media ability

GET /protocol/media/ability

获取协议媒体能力集（RTSP/RTMP/SRT/NDI/FullNDI/Dante/ONVIF 等支持的参数范围）

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "message": "",
  "data": {
    "rtsp": 1,
    "rtmp": 1,
    "srt": 1,
    "ndi": 1,
    "dante": 1,
    "full_ndi": 0,
    "onvif": 1
  }
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST rtsp get

POST /protocol/media/rtsp/get

RTSP channel parameters get. Includes per-channel parameters (audio, enable, enableFixed, streamName) and global parameters (port, rtspOverHttpEnable, rtspOverHttpPort, authEnable, username, password). For global parameters, the `id` field in the request body is ignored.

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "audio",
        "enable",
        "enableFixed",
        "streamName",
        "port",
        "rtspOverHttpEnable",
        "rtspOverHttpPort",
        "authEnable",
        "username",
        "password"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[RTSP Key](#schemartsp key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | audio |
| » keys | enable |
| » keys | enableFixed |
| » keys | streamName |
| » keys | port |
| » keys | rtspOverHttpEnable |
| » keys | rtspOverHttpPort |
| » keys | authEnable |
| » keys | username |
| » keys | password |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "audio": 6,
    "enable": true,
    "enable_fixed": false,
    "stream_name": "ch1",
    "port": 554,
    "rtsp_over_http_enable": false,
    "rtsp_over_http_port": 8080,
    "auth_enable": false,
    "username": "",
    "password": ""
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST rtsp set

POST /protocol/media/rtsp/set

RTSP channel parameters set. Includes per-channel parameters (audio, enable, enableFixed, streamName) and global parameters (port, rtspOverHttpEnable, rtspOverHttpPort, authEnable, username, password). For global parameters, the `id` field in the request body is ignored. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "streamName",
            "value": "ch1"
        },
        {
            "key": "port",
            "value": 554
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | audio |
| » key | enable |
| » key | enableFixed |
| » key | streamName |
| » key | port |
| » key | rtspOverHttpEnable |
| » key | rtspOverHttpPort |
| » key | authEnable |
| » key | username |
| » key | password |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET rtsp ability

GET /protocol/media/rtsp/ability

获取 RTSP 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 RTSP Key 枚举所列的全部可配置字段（audio / enable / enableFixed / streamName / port / rtspOverHttpEnable / rtspOverHttpPort / authEnable / username / password）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "enableFixed",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "audio",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": -1,
      "feature": {
        "options": [
          { "label": "None", "value": -1 },
          { "label": "AAC", "value": 6 }
        ]
      }
    },
    {
      "key": "streamName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "port",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 554,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "rtspOverHttpEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "rtspOverHttpPort",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 8000,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "authEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "username",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "password",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST rtmp get

POST /protocol/media/rtmp/get

RTMP channel parameters get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "video",
        "audio",
        "url",
        "key",
        "authEnable",
        "username",
        "password",
        "enable",
        "enableFixed"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[RTMP Key](#schemartmp key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | audio |
| » keys | authEnable |
| » keys | enable |
| » keys | enableFixed |
| » keys | key |
| » keys | password |
| » keys | url |
| » keys | username |
| » keys | video |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "enable": true,
    "enable_fixed": false,
    "video": 1,
    "audio": 6,
    "url": "rtmp://server.example.com/live/stream",
    "auth_enable": false,
    "username": "",
    "password": "",
    "status": "ok"
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST rtmp set

POST /protocol/media/rtmp/set

RTMP channel parameters set. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "url",
            "value": "rtmp://server/live/stream"
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | audio |
| » key | authEnable |
| » key | enable |
| » key | enableFixed |
| » key | key |
| » key | password |
| » key | url |
| » key | username |
| » key | video |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET rtmp ability

GET /protocol/media/rtmp/ability

获取 RTMP 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 RTMP Key 枚举所列的全部可配置字段（audio / authEnable / enable / enableFixed / key / password / url / username / video）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "enableFixed",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "video",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "options": [
          { "label": "None", "value": -1 },
          { "label": "Main", "value": 1 }
        ]
      }
    },
    {
      "key": "audio",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": -1,
      "feature": {
        "options": [
          { "label": "None", "value": -1 },
          { "label": "AAC", "value": 6 }
        ]
      }
    },
    {
      "key": "url",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "key",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "authEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "username",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "password",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST srt get

POST /protocol/media/srt/get

SRT channel parameters get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "enable",
        "enableFixed",
        "mode",
        "hostname",
        "port",
        "latency",
        "aesEnable",
        "aesMode",
        "password",
        "streamid",
        "audio"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[SRT Key](#schemasrt key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | aesEnable |
| » keys | aesMode |
| » keys | audio |
| » keys | enable |
| » keys | enableFixed |
| » keys | hostname |
| » keys | latency |
| » keys | mode |
| » keys | password |
| » keys | port |
| » keys | streamid |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "enable": true,
    "enable_fixed": false,
    "mode": 1,
    "streamid": "ch1",
    "hostname": "",
    "port": 9000,
    "password": "",
    "aes_mode": 16,
    "latency": 120,
    "status": "ok"
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST srt set

POST /protocol/media/srt/set

SRT channel parameters set. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "port",
            "value": 9000
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | aesEnable |
| » key | aesMode |
| » key | audio |
| » key | enable |
| » key | enableFixed |
| » key | hostname |
| » key | latency |
| » key | mode |
| » key | password |
| » key | port |
| » key | streamid |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET srt ability

GET /protocol/media/srt/ability

获取 SRT 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 SRT Key 枚举所列的全部可配置字段（aesEnable / aesMode / audio / enable / enableFixed / hostname / latency / mode / password / port / streamid）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "enableFixed",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "audio",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": -1,
      "feature": {
        "options": [
          { "label": "None", "value": -1 },
          { "label": "AAC", "value": 6 }
        ]
      }
    },
    {
      "key": "mode",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "options": [
          { "label": "Caller", "value": 1 },
          { "label": "Host", "value": 2 }
        ]
      }
    },
    {
      "key": "aesEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "aesMode",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 16,
      "feature": {
        "options": [
          { "label": "AES-16", "value": 16 },
          { "label": "AES-24", "value": 24 },
          { "label": "AES-32", "value": 32 }
        ]
      }
    },
    {
      "key": "latency",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 120,
      "feature": {
        "min": 0,
        "max": 1000,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "port",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 9000,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "hostname",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "streamid",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "password",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST ndi get

POST /protocol/media/ndi/get

NDI channel parameters get. Includes per-channel parameters (enable, enableFixed, streamName, streamNameSuffix) and global parameters (discoveryEnable, discoveryServer, groupName, multicastEnable, multicastIp, multicastNetmask, multicastTtl, ndiAudio, tpm). For global parameters, the `id` field in the request body is ignored.

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "enable",
        "enableFixed",
        "streamName",
        "streamNameSuffix",
        "discoveryEnable",
        "discoveryServer",
        "groupName",
        "multicastEnable",
        "multicastIp",
        "multicastNetmask",
        "multicastTtl",
        "ndiAudio",
        "tpm"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[NDI Key](#schemandi key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | enable |
| » keys | enableFixed |
| » keys | streamName |
| » keys | streamNameSuffix |
| » keys | discoveryEnable |
| » keys | discoveryServer |
| » keys | groupName |
| » keys | multicastEnable |
| » keys | multicastIp |
| » keys | multicastNetmask |
| » keys | multicastTtl |
| » keys | ndiAudio |
| » keys | tpm |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "enable": true,
    "enable_fixed": false,
    "stream_name": "ch1",
    "stream_name_suffix": "",
    "discovery_enable": true,
    "discovery_server": "",
    "group_name": "public",
    "multicast_enable": false,
    "multicast_ip": "",
    "multicast_netmask": "",
    "multicast_ttl": 4,
    "ndi_audio": -1,
    "tpm": false
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST ndi set

POST /protocol/media/ndi/set

NDI channel parameters set. Includes per-channel parameters (enable, enableFixed, streamName, streamNameSuffix) and global parameters (discoveryEnable, discoveryServer, groupName, multicastEnable, multicastIp, multicastNetmask, multicastTtl, ndiAudio, tpm). For global parameters, the `id` field in the request body is ignored. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "streamName",
            "value": "Close-up"
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | enable |
| » key | enableFixed |
| » key | streamName |
| » key | streamNameSuffix |
| » key | discoveryEnable |
| » key | discoveryServer |
| » key | groupName |
| » key | multicastEnable |
| » key | multicastIp |
| » key | multicastNetmask |
| » key | multicastTtl |
| » key | ndiAudio |
| » key | tpm |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET ndi ability

GET /protocol/media/ndi/ability

获取 NDI 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 NDI Key 枚举所列的全部可配置字段（enable / enableFixed / streamName / streamNameSuffix / discoveryEnable / discoveryServer / groupName / multicastEnable / multicastIp / multicastNetmask / multicastTtl / ndiAudio / tpm）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "enableFixed",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "streamName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "streamNameSuffix",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "discoveryEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "discoveryServer",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "groupName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "multicastEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "multicastIp",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "multicastNetmask",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "multicastTtl",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 4,
      "feature": {
        "min": 1,
        "max": 255,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "ndiAudio",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": -1,
      "feature": {
        "options": [
          { "label": "None", "value": -1 },
          { "label": "AAC", "value": 6 }
        ]
      }
    },
    {
      "key": "tpm",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "min": 0,
        "max": 100,
        "step": 1,
        "input": 1
      }
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST full ndi get

POST /protocol/media/full-ndi/get

Full NDI channel parameters get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "channelName",
        "channelNameSuffix",
        "deviceName",
        "enable",
        "encodeQuality",
        "dhcp",
        "ip",
        "netmask",
        "gateway",
        "dns",
        "staticIp",
        "fallbackIp",
        "fallbackNetmask",
        "dynamic",
        "multicastEnable",
        "multicastIp",
        "multicastNetmask",
        "ttl",
        "groupName"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[Full NDI Key](#schemafull ndi key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | channelName |
| » keys | channelNameSuffix |
| » keys | deviceName |
| » keys | dhcp |
| » keys | dns |
| » keys | dynamic |
| » keys | enable |
| » keys | encodeQuality |
| » keys | fallbackIp |
| » keys | fallbackNetmask |
| » keys | gateway |
| » keys | groupName |
| » keys | ip  |
| » keys | multicastEnable |
| » keys | multicastIp |
| » keys | multicastNetmask |
| » keys | netmask |
| » keys | staticIp |
| » keys | ttl |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "enable": true,
    "group_name": "public",
    "device_name": "BDG-XL-001",
    "channel_name": "Channel 1",
    "encode_quality": 80,
    "static_ip": "",
    "gateway": "",
    "netmask": "",
    "dns": "",
    "dynamic": true,
    "multicast_enable": false,
    "multicast_ip": "",
    "multicast_netmask": "",
    "ttl": 4
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST full ndi set

POST /protocol/media/full-ndi/set

Full NDI channel parameters set. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "channelName",
            "value": "FullNDI"
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | channelName |
| » key | channelNameSuffix |
| » key | deviceName |
| » key | dhcp |
| » key | dns |
| » key | dynamic |
| » key | enable |
| » key | encodeQuality |
| » key | fallbackIp |
| » key | fallbackNetmask |
| » key | gateway |
| » key | groupName |
| » key | ip  |
| » key | multicastEnable |
| » key | multicastIp |
| » key | multicastNetmask |
| » key | netmask |
| » key | staticIp |
| » key | ttl |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET full ndi ability

GET /protocol/media/full-ndi/ability

获取 Full NDI 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 Full NDI Key 枚举所列的全部可配置字段（channelName / channelNameSuffix / deviceName / dhcp / dns / dynamic / enable / encodeQuality / fallbackIp / fallbackNetmask / gateway / groupName / ip / multicastEnable / multicastIp / multicastNetmask / netmask / staticIp / ttl）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "encodeQuality",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 80,
      "feature": {
        "min": 0,
        "max": 100,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "ttl",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 4,
      "feature": {
        "min": 1,
        "max": 255,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "multicastEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "multicastIp",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "multicastNetmask",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "deviceName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "channelName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "channelNameSuffix",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "groupName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "dhcp",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "dynamic",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    },
    {
      "key": "ip",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "netmask",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "gateway",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "dns",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "staticIp",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "fallbackIp",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    },
    {
      "key": "fallbackNetmask",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": ""
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST dante get

POST /protocol/media/dante-av-h/get

Dante AV-H channel parameters get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "enable"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[Dante Key](#schemadante key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | enable |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "enable": false
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST dante set

POST /protocol/media/dante-av-h/set

Dante AV-H channel parameters set. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "enable",
            "value": true
        },
        {
            "key": "enable",
            "value": true
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | enable |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET dante ability

GET /protocol/media/dante-av-h/ability

获取 Dante AV-H 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，目前仅包含 Dante Key 枚举所列的 enable 字段。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST onvif get

POST /protocol/media/onvif/get

ONVIF channel parameters get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "enable",
        "port"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[ONVIF Key](#schemaonvif key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | enable |
| » keys | port |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "index": 0,
    "enable": false,
    "port": 80
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST onvif set

POST /protocol/media/onvif/set

ONVIF channel parameters set. 支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "port",
            "value": 80
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | enable |
| » key | port |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET onvif ability

GET /protocol/media/onvif/ability

获取 ONVIF 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 ONVIF Key 枚举所列的全部可配置字段（enable / port）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "port",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 8080,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1,
        "input": 1
      }
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

# PBC-120/Media Parameter

## GET mpp ability

GET /mpp/ability

获取 MPP 子模块支持情况（video / audio / hdmi / pip）。data 为数组，索引 0 表示各子模块是否在当前设备上支持（0 不支持 / 1 支持），索引 1 表示各子模块在当前设备上是否提供独立 ability 接口（0 不提供 / 1 提供）。具体可配置项范围请参考各子模块 ability 接口（`/mpp/video/ability`、`/mpp/audio/ability`、`/hdmi/ability`、`/pip/ability`）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "message": "",
  "data": [
    {"video": 1, "audio": 1, "hdmi": 1, "pip": 1},
    {"hdmi": 0, "pip": 0}
  ]
}
```

## POST video get

POST /mpp/video/get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "codec",
        "profile",
        "size",
        "fps",
        "gop",
        "bitrate",
        "rc",
        "BitrateProfile",
        "QuantFactorI",
        "QuantFactorP"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[Video Key](#schemavideo key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | BitrateProfile |
| » keys | QuantFactorI |
| » keys | QuantFactorP |
| » keys | bitrate |
| » keys | codec |
| » keys | fps |
| » keys | gop |
| » keys | profile |
| » keys | rc  |
| » keys | size |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "id": 0,
    "size": 0,
    "fps": 0,
    "codec": 0,
    "profile": 0,
    "bitrate": 0,
    "rc": 0,
    "gop": 0
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST video set

POST /mpp/video/set

支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "bitrate",
            "value": 20000
        },
        {
            "key": "fps",
            "value": 30
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | BitrateProfile |
| » key | QuantFactorI |
| » key | QuantFactorP |
| » key | bitrate |
| » key | codec |
| » key | fps |
| » key | gop |
| » key | profile |
| » key | rc  |
| » key | size |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET video ability

GET /mpp/video/ability

获取 video 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 Video Key 枚举所列的全部可配置字段（codec / profile / size / fps / gop / bitrate / rc / BitrateProfile / QuantFactorI / QuantFactorP）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "codec",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": -1,
      "feature": {
        "options": [
          {
            "label": "H.264",
            "value": 0
          },
          {
            "label": "H.265",
            "value": 1
          }
        ]
      }
    },
    {
      "key": "profile",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "Baseline",
            "value": 0
          },
          {
            "label": "Main",
            "value": 1
          },
          {
            "label": "High",
            "value": 2
          }
        ]
      }
    },
    {
      "key": "size",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "1920x1080",
            "value": 0
          },
          {
            "label": "1280x720",
            "value": 1
          }
        ]
      }
    },
    {
      "key": "fps",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "25",
            "value": 0
          },
          {
            "label": "30",
            "value": 1
          },
          {
            "label": "50",
            "value": 2
          },
          {
            "label": "60",
            "value": 3
          }
        ]
      }
    },
    {
      "key": "gop",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 60,
      "feature": {
        "min": 1,
        "max": 300,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "bitrate",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 4096,
      "feature": {
        "min": 100,
        "max": 20000,
        "step": 100,
        "input": 1
      }
    },
    {
      "key": "rc",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "CBR",
            "value": 0
          },
          {
            "label": "VBR",
            "value": 1
          },
          {
            "label": "FIXQP",
            "value": 2
          }
        ]
      }
    },
    {
      "key": "BitrateProfile",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "QuantFactorI",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 20,
      "feature": {
        "min": 1,
        "max": 51,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "QuantFactorP",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 20,
      "feature": {
        "min": 1,
        "max": 51,
        "step": 1,
        "input": 1
      }
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST audio get

POST /mpp/audio/get

获取音频输出配置

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "codec",
        "audioInput",
        "channels",
        "samplerate",
        "bitrate",
        "volume",
        "mute"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[Audio Key](#schemaaudio key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | audioInput |
| » keys | bitrate |
| » keys | channels |
| » keys | codec |
| » keys | mute |
| » keys | samplerate |
| » keys | volume |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "id": 0,
    "samplerate": 0,
    "channels": 0,
    "volume": 0,
    "volumeId": 0
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST audio set

POST /mpp/audio/set

支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "volume",
            "value": 79
        },
        {
            "key": "codec",
            "value": 0
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | audioInput |
| » key | bitrate |
| » key | channels |
| » key | codec |
| » key | mute |
| » key | samplerate |
| » key | volume |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET audio ability

GET /mpp/audio/ability

获取 audio 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 Audio Key 枚举所列的全部可配置字段（codec / audioInput / channels / samplerate / bitrate / volume / mute）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "codec",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": -1,
      "feature": {
        "options": [
          {
            "label": "AAC",
            "value": 6
          },
          {
            "label": "G711A",
            "value": 7
          }
        ]
      }
    },
    {
      "key": "audioInput",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "channels",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "options": [
          {
            "label": "Mono",
            "value": 1
          },
          {
            "label": "Stereo",
            "value": 2
          }
        ]
      }
    },
    {
      "key": "samplerate",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 48000,
      "feature": {
        "options": [
          {
            "label": "8K",
            "value": 8000
          },
          {
            "label": "16K",
            "value": 16000
          },
          {
            "label": "32K",
            "value": 32000
          },
          {
            "label": "44.1K",
            "value": 44100
          },
          {
            "label": "48K",
            "value": 48000
          }
        ]
      }
    },
    {
      "key": "bitrate",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 128,
      "feature": {
        "min": 16,
        "max": 320,
        "step": 16,
        "input": 1
      }
    },
    {
      "key": "volume",
      "component": "Slider",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 50,
      "feature": {
        "min": 0,
        "max": 100,
        "step": 1,
        "input": 1
      }
    },
    {
      "key": "mute",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST hdmi get

POST /hdmi/get

获取 HDMI 输出参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "hdmi_output",
        "hdmi_format",
        "hdmi_fps",
        "hdmi_cs"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "hdmi_output", "value": 0},
    {"key": "hdmi_format", "value": 0},
    {"key": "hdmi_fps", "value": 0},
    {"key": "hdmi_cs", "value": 0}
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [HDMI](#schemahdmi) | true | none |     | none |
| »» hdmiCs | integer | true | none |     | Color Space |
| »» hdmiFormat | integer | true | none |     | Resolution |
| »» hdmiFps | integer | true | none |     | Frame Rate |
| »» hdmiOutput | integer | true | none |     | Output Channel |
| » message | string | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| hdmiCs | 0   |
| hdmiCs | 1   |
| hdmiCs | 2   |
| hdmiFormat | 0   |
| hdmiFormat | 1   |
| hdmiFormat | 2   |
| hdmiFormat | 3   |
| hdmiFps | 0   |
| hdmiFps | 1   |
| hdmiFps | 2   |
| hdmiFps | 3   |
| hdmiFps | 4   |
| hdmiFps | 5   |
| hdmiFps | 7   |
| hdmiOutput | 0   |
| hdmiOutput | 1   |

## POST hdmi set

POST /hdmi/set

设置 HDMI 输出参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "hdmi_output",
            "value": 1
        },
        {
            "key": "hdmi_format",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## GET hdmi ability

GET /hdmi/ability

获取 HDMI 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 HDMI 全部可配置字段（hdmiCs / hdmiFormat / hdmiFps / hdmiOutput）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "hdmiCs",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "RGB",
            "value": 0
          },
          {
            "label": "YCbCr444",
            "value": 1
          },
          {
            "label": "YCbCr422",
            "value": 2
          }
        ]
      }
    },
    {
      "key": "hdmiFormat",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "1080p",
            "value": 0
          },
          {
            "label": "1080i",
            "value": 1
          },
          {
            "label": "720p",
            "value": 2
          },
          {
            "label": "4K",
            "value": 3
          }
        ]
      }
    },
    {
      "key": "hdmiFps",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "24",
            "value": 0
          },
          {
            "label": "25",
            "value": 1
          },
          {
            "label": "30",
            "value": 2
          },
          {
            "label": "50",
            "value": 3
          },
          {
            "label": "60",
            "value": 4
          }
        ]
      }
    },
    {
      "key": "hdmiOutput",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "Off",
            "value": 0
          },
          {
            "label": "On",
            "value": 1
          }
        ]
      }
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST pip get

POST /pip/get

获取 PIP 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "Pip",
        "PipPosition",
        "PipSize",
        "PipSource"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "Pip", "value": 0},
    {"key": "PipPosition", "value": 0},
    {"key": "PipSize", "value": 0},
    {"key": "PipSource", "value": 0}
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [PIP](#schemapip) | true | none |     | none |
| »» Pip | [Enable](#schemaenable) | true | none |     | none |
| »» PipPosition | integer | true | none |     | none |
| »» PipSize | integer | true | none |     | none |
| »» PipSource | integer | true | none |     | none |
| » message | string | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| Pip | 0   |
| Pip | 1   |
| PipPosition | 0   |
| PipPosition | 1   |
| PipPosition | 2   |
| PipPosition | 3   |
| PipSize | 0   |
| PipSize | 1   |
| PipSize | 2   |
| PipSource | 0   |
| PipSource | 1   |

## POST pip set

POST /pip/set

设置 PIP 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "Pip",
            "value": 1
        },
        {
            "key": "PipPosition",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

## GET pip ability

GET /pip/ability

获取 PIP 能力集。返回与 [WEB_API_GetAbility](#) 相同结构的 ability 数组，覆盖 PIP 全部可配置字段（Pip / PipPosition / PipSize / PipSource）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "Pip",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "PipPosition",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "TopLeft",
            "value": 0
          },
          {
            "label": "TopRight",
            "value": 1
          },
          {
            "label": "BottomLeft",
            "value": 2
          },
          {
            "label": "BottomRight",
            "value": 3
          }
        ]
      }
    },
    {
      "key": "PipSize",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {
            "label": "Small",
            "value": 0
          },
          {
            "label": "Medium",
            "value": 1
          },
          {
            "label": "Large",
            "value": 2
          }
        ]
      }
    },
    {
      "key": "PipSource",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    }
  ],
  "message": ""
}
```

## POST 3g sdi get

POST /system/sdi-3g-get

获取 3G SDI 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。3G SDI 与 3 路 4K 视频互斥（weight=180 时不能开启 3G SDI）。

> Body 请求参数

```json
{
    "keys": [
        "enable",
        "weight"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "enable", "value": 0, "enabled": true},
    {"key": "weight", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST 3g sdi set

POST /system/sdi-3g-set

设置 3G SDI 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "enable",
            "value": 1
        },
        {
            "key": "weight",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET 3g sdi ability

GET /sdi-3g/ability

获取 3G SDI 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。enable 为开关；weight 为当前 3G SDI 占用权重（0 或 180）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "weight",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "min": 0,
        "max": 180,
        "step": 1
      }
    }
  ],
  "message": ""
}
```

## POST stream screensaver get

POST /protocol/media/stream-screensaver

获取流屏保参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "customColor",
        "enable",
        "imgName",
        "screensaverMode"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "customColor", "value": 16777216, "enabled": true},
    {"key": "enable", "value": 0, "enabled": true},
    {"key": "imgName", "value": "string", "enabled": true},
    {"key": "screensaverMode", "value": 1, "enabled": true}
  ],
  "message": ""
}
```

## POST stream screensaver set

POST /protocol/media/stream-screensaver

设置流屏保参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "screensaverMode",
            "value": 1
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## POST teleconvert mode get

POST /system/sensor-crop/get

获取 Teleconvert Mode 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "sensorCrop"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "sensorCrop", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST teleconvert mode set

POST /system/sensor-crop/set

设置 Teleconvert Mode 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "sensorCrop",
            "value": 1
        },
        {
            "key": "sensorCrop",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

# PBC-120/Control

## GET protocol control ability

GET /protocol/control/ability

获取控制协议子模块支持情况（visca / viscaOverIp / freed / ir / pelcoPD / usb / tally / serialIn / serialOut 等）。data 字段为各子模块的支持标志位（0 不支持 / 1 支持），具体可配置项范围请参考各子模块 get 接口。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "message": "",
  "data": {
    "visca": 1,
    "viscaOverIp": 1,
    "freed": 1,
    "ir": 1,
    "pelcoPD": 1,
    "usb": 1,
    "tally": 1,
    "serialIn": 1,
    "serialOut": 1
  }
}
```

## POST visca passthrough get

POST /protocol/control/visca

获取 Visca Passthrough 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "port",
        "enable",
        "protocol_type",
        "work_mode",
        "address",
        "Visca_addr"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "port", "value": 1, "enabled": true},
    {"key": "enable", "value": true, "enabled": true},
    {"key": "protocol_type", "value": 1, "enabled": true},
    {"key": "work_mode", "value": 1, "enabled": true},
    {"key": "address", "value": "192.168.0.1", "enabled": true},
    {"key": "Visca_addr", "value": 1, "enabled": true}
  ],
  "message": ""
}
```

## POST visca passthrough set

POST /protocol/control/visca

设置 Visca Passthrough 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "port",
            "value": 52380
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET visca passthrough ability

GET /protocol/control/visca/ability

获取 visca passthrough 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "protocol_type",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "Visca", "value": 0},
          {"label": "PelcoD", "value": 1},
          {"label": "PelcoP", "value": 2}
        ]
      }
    },
    {
      "key": "work_mode",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "透传", "value": 0},
          {"label": "转换", "value": 1}
        ]
      }
    },
    {
      "key": "address",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "min": 0,
        "max": 7,
        "step": 1
      }
    },
    {
      "key": "Visca_addr",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "options": [
          {"label": "0", "value": 0},
          {"label": "1", "value": 1},
          {"label": "2", "value": 2},
          {"label": "3", "value": 3},
          {"label": "4", "value": 4},
          {"label": "5", "value": 5},
          {"label": "6", "value": 6},
          {"label": "7", "value": 7}
        ]
      }
    },
    {
      "key": "baudrate",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 2,
      "feature": {
        "options": [
          {"label": "2400", "value": 0},
          {"label": "4800", "value": 1},
          {"label": "9600", "value": 2},
          {"label": "19200", "value": 3},
          {"label": "38400", "value": 4}
        ]
      }
    },
    {
      "key": "port",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 52381,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1
      }
    },
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    }
  ],
  "message": ""
}
```

## POST visca over ip get

POST /protocol/control/viscaOverIp

获取 Visca Over IP 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "port",
        "enable"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "port", "value": 1, "enabled": true},
    {"key": "enable", "value": true, "enabled": true}
  ],
  "message": ""
}
```

## POST visca over ip set

POST /protocol/control/viscaOverIp

设置 Visca Over IP 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "port",
            "value": 52381
        },
        {
            "key": "enable",
            "value": 1
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET visca over ip ability

GET /protocol/control/viscaOverIp/ability

获取 visca over ip 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "port",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 52381,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1
      }
    },
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    }
  ],
  "message": ""
}
```

## POST input serial port get

POST /protocol/control/serial/in

获取 Input Serial Port 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "ComModeIn": 0,
    "keys": [
        "ProtocolTypeIn",
        "BaudrateIn"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "ProtocolTypeIn", "value": 0, "enabled": true},
    {"key": "BaudrateIn", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST input serial port set

POST /protocol/control/serial/in

设置 Input Serial Port 参数。支持一次下发多组 key + value。ComModeIn 用于选择串口模式。

> Body 请求参数

```json
{
    "ComModeIn": 0,
    "id": 0,
    "data": [
        {
            "key": "BaudrateIn",
            "value": 3
        },
        {
            "key": "ProtocolTypeIn",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET input serial port ability

GET /protocol/control/serial/in/ability

获取 input serial port 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。ProtocolType/Baudrate 由 system 全局 ability 加载（`__SYSTEM_GetAbilityEnum("ProtocolTypeIn"/"BaudrateIn")`）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "ProtocolTypeIn",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "Visca", "value": 0},
          {"label": "PelcoD", "value": 1},
          {"label": "PelcoP", "value": 2}
        ]
      }
    },
    {
      "key": "BaudrateIn",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 2,
      "feature": {
        "options": [
          {"label": "2400", "value": 0},
          {"label": "4800", "value": 1},
          {"label": "9600", "value": 2},
          {"label": "19200", "value": 3},
          {"label": "38400", "value": 4}
        ]
      }
    }
  ],
  "message": ""
}
```

## POST freed get

POST /protocol/control/FreeD

获取 FreeD 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "enable",
        "ip_addr",
        "port",
        "carema_id"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "enable", "value": 0, "enabled": true},
    {"key": "ip_addr", "value": "192.168.0.1", "enabled": true},
    {"key": "port", "value": 1, "enabled": true},
    {"key": "carema_id", "value": 1, "enabled": true}
  ],
  "message": ""
}
```

## POST freed set

POST /protocol/control/FreeD

设置 FreeD 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "port",
            "value": 40000
        },
        {
            "key": "ip_addr",
            "value": "192.168.0.1"
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET freed ability

GET /protocol/control/FreeD/ability

获取 FreeD 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "enable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "ip_addr",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "192.168.0.1",
      "feature": {
        "pattern": "^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$"
      }
    },
    {
      "key": "port",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 40000,
      "feature": {
        "min": 1,
        "max": 65535,
        "step": 1
      }
    },
    {
      "key": "carema_id",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "min": 1,
        "max": 255,
        "step": 1
      }
    }
  ],
  "message": ""
}
```

## POST ir control get

POST /protocol/control/Ir

获取 IR Control 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "bEnable",
        "Ir_addr"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "bEnable", "value": true, "enabled": true},
    {"key": "Ir_addr", "value": 1, "enabled": true}
  ],
  "message": ""
}
```

## POST ir control set

POST /protocol/control/Ir

设置 IR Control 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "Ir_addr",
            "value": 3
        },
        {
            "key": "bEnable",
            "value": true
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET ir control ability

GET /protocol/control/Ir/ability

获取 IR Control 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。Ir_addr 由 system 全局 ability 加载（`__SYSTEM_GetAbilityEnum("IrAddr")`）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "bEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": false
    },
    {
      "key": "Ir_addr",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1,
      "feature": {
        "options": [
          {"label": "0", "value": 0},
          {"label": "1", "value": 1},
          {"label": "2", "value": 2},
          {"label": "3", "value": 3}
        ]
      }
    }
  ],
  "message": ""
}
```

## POST pelcopd get

POST /protocol/control/PelcoPD

获取 PelcoP/D 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "PelcoP_addr",
        "PelcoD_addr"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "PelcoP_addr", "value": 31, "enabled": true},
    {"key": "PelcoD_addr", "value": 255, "enabled": true}
  ],
  "message": ""
}
```

## POST pelcopd set

POST /protocol/control/PelcoPD

设置 PelcoP/D 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "PelcoD_addr",
            "value": 158
        },
        {
            "key": "PelcoP_addr",
            "value": 31
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET pelcopd ability

GET /protocol/control/PelcoPD/ability

获取 PelcoP/D 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "PelcoP_addr",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 31,
      "feature": {
        "min": 0,
        "max": 31,
        "step": 1
      }
    },
    {
      "key": "PelcoD_addr",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 255,
      "feature": {
        "min": 0,
        "max": 255,
        "step": 1
      }
    }
  ],
  "message": ""
}
```

## POST usb get

POST /protocol/control/usb

获取 USB 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "usbDevice"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "usbDevice", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST usb set

POST /protocol/control/usb

设置 USB 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "usbDevice",
            "value": 1
        },
        {
            "key": "usbDevice",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET usb ability

GET /protocol/control/usb/ability

获取 USB 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "usbDevice",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "无", "value": 0},
          {"label": "USB 摄像头", "value": 1}
        ]
      }
    }
  ],
  "message": ""
}
```

## POST tally get

POST /protocol/control/tally

获取 Tally 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "customRgb",
        "ndiTally",
        "tallyResetStatus",
        "trackTally"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "customRgb", "value": 16777216, "enabled": true},
    {"key": "ndiTally", "value": 0, "enabled": true},
    {"key": "tallyResetStatus", "value": 0, "enabled": true},
    {"key": "trackTally", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST tally set

POST /protocol/control/tally

设置 Tally 参数。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "tallyResetStatus",
            "value": 1
        },
        {
            "key": "ndiTally",
            "value": 0
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET tally ability

GET /protocol/control/tally/ability

获取 Tally 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "customRgb",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 16777216,
      "feature": {
        "min": 0,
        "max": 16777215,
        "step": 1
      }
    },
    {
      "key": "ndiTally",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "tallyResetStatus",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "trackTally",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    }
  ],
  "message": ""
}
```

# PBC-120/Basic

## GET ping

GET /ping

Check if web service is normal

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "pong",
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST export-log

POST /system/export-log

Export logs, return binary stream of log files, need to save manually

> 返回示例

> 200 Response

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST export-config

POST /system/export-config

Export config

> 返回示例

> 200 Response

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET upgrade protect

GET /system/upgrade/get-protect

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "label": "NETWORK",
      "value": 1
    },
    {
      "label": "WEB_USER",
      "value": 4
    },
    {
      "label": "ONVIF_USER",
      "value": 8
    },
    {
      "label": "IMAGE",
      "value": 16
    },
    {
      "label": "TRACK_AREA",
      "value": 64
    },
    {
      "label": "Track",
      "value": 128
    },
    {
      "label": "MPP_VIDEO",
      "value": 256
    },
    {
      "label": "PRESET",
      "value": 512
    },
    {
      "label": "CMS",
      "value": 1024
    },
    {
      "label": "System",
      "value": 2048
    },
    {
      "label": "ControlProtocol",
      "value": 4096
    },
    {
      "label": "Audio",
      "value": 8192
    },
    {
      "label": "LiveMedia",
      "value": 16384
    },
    {
      "label": "Output",
      "value": 32768
    },
    {
      "label": "Audio",
      "value": 65536
    },
    {
      "label": "LiveMediaService",
      "value": 131072
    },
    {
      "label": "Ntpd",
      "value": 262144
    },
    {
      "label": "OF_AP",
      "value": 524288
    },
    {
      "label": "OF_CC",
      "value": 1048576
    },
    {
      "label": "OF_DA",
      "value": 2097152
    },
    {
      "label": "OF_E",
      "value": 4194304
    },
    {
      "label": "OF_MC",
      "value": 8388608
    },
    {
      "label": "OF_S",
      "value": 16777216
    },
    {
      "label": "OF_Users",
      "value": 33554432
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [object] | true | none |     | none |
| »» label | string | true | none |     | none |
| »» value | integer | true | none |     | none |
| » message | string | true | none |     | none |

## POST upgrade-config

POST /system/upgrade-config

Import config

> Body 请求参数

```json
{
    "filename": "",
    "bitmap": 0
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » filename | body | string | 是   |     | The name of the configuration zip file uploaded to the device |
| » bitmap | body | [Upgrade Protect](#schemaupgrade protect) | 是   |     | Fields retained when importing settings. The sum of the corresponding values |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » bitmap | 1   |
| » bitmap | 4   |
| » bitmap | 8   |
| » bitmap | 16  |
| » bitmap | 64  |
| » bitmap | 128 |
| » bitmap | 256 |
| » bitmap | 512 |
| » bitmap | 1024 |
| » bitmap | 2048 |
| » bitmap | 4096 |
| » bitmap | 8192 |
| » bitmap | 16384 |
| » bitmap | 32768 |
| » bitmap | 65536 |
| » bitmap | 131072 |
| » bitmap | 262144 |
| » bitmap | 524288 |
| » bitmap | 1048576 |
| » bitmap | 2097152 |
| » bitmap | 4194304 |
| » bitmap | 8388608 |
| » bitmap | 16777216 |
| » bitmap | 33554432 |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST upgrade firmware prepare

POST /system/upgrade-firmware/prepare

Call this API before uploading firmware, notify device to enter upgrade state

> Body 请求参数

```json
{
    "bitmap": 0
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » bitmap | body | [Upgrade Protect](#schemaupgrade protect) | 是   |     | Fields retained when importing settings. The sum of the corresponding values |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » bitmap | 1   |
| » bitmap | 4   |
| » bitmap | 8   |
| » bitmap | 16  |
| » bitmap | 64  |
| » bitmap | 128 |
| » bitmap | 256 |
| » bitmap | 512 |
| » bitmap | 1024 |
| » bitmap | 2048 |
| » bitmap | 4096 |
| » bitmap | 8192 |
| » bitmap | 16384 |
| » bitmap | 32768 |
| » bitmap | 65536 |
| » bitmap | 131072 |
| » bitmap | 262144 |
| » bitmap | 524288 |
| » bitmap | 1048576 |
| » bitmap | 2097152 |
| » bitmap | 4194304 |
| » bitmap | 8388608 |
| » bitmap | 16777216 |
| » bitmap | 33554432 |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST upgrade firmware chunk

POST /system/upgrade-firmware/chunk

After device enters upgrade state, upload firmware chunks in order

> Body 请求参数

```yaml
fileName: ""
fileSize: ""
chunkSize: ""
currentSize: ""
index: ""
data: ""
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » fileName | body | string | 否   |     | Filename |
| » fileSize | body | string | 否   |     | File size |
| » chunkSize | body | string | 否   |     | Chunk Size |
| » currentSize | body | string | 否   |     | Current Chunk Size |
| » index | body | string | 否   |     | Current Chunk Index |
| » data | body | string(binary) | 否   |     | Current Chunk File Data |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST upgrade firmware confirm

POST /system/upgrade-firmware/confirm

After firmware chunks uploaded, call this API to confirm upgrade

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST upgrade algorithm confirm

POST /system/upgrade-algorithm/confirm

After firmware chunks uploaded, call this API to confirm upgrade

> Body 请求参数

```json
{
    "filename": ""
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » filename | body | string | 是   |     | 已上传的用于升级算法的算法包文件名 |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST factory-reset

POST /system/factory-reset

Factory reset

> Body 请求参数

```json
{
    "bitmap": 0
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » bitmap | body | [Upgrade Protect](#schemaupgrade protect) | 是   |     | Fields retained when factory reset. The sum of the corresponding values |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » bitmap | 1   |
| » bitmap | 4   |
| » bitmap | 8   |
| » bitmap | 16  |
| » bitmap | 64  |
| » bitmap | 128 |
| » bitmap | 256 |
| » bitmap | 512 |
| » bitmap | 1024 |
| » bitmap | 2048 |
| » bitmap | 4096 |
| » bitmap | 8192 |
| » bitmap | 16384 |
| » bitmap | 32768 |
| » bitmap | 65536 |
| » bitmap | 131072 |
| » bitmap | 262144 |
| » bitmap | 524288 |
| » bitmap | 1048576 |
| » bitmap | 2097152 |
| » bitmap | 4194304 |
| » bitmap | 8388608 |
| » bitmap | 16777216 |
| » bitmap | 33554432 |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET reboot

GET /reboot

Device reboot, device will reboot immediately after calling

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST restart video

POST /restart-video

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET title

GET /system/title

Get camera name

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "PBC-120",
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET camera-type

GET /system/camera-type

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "cid": "COMMON",
    "pid": "PBC-120"
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

# PBC-120/System

## POST language get

POST /system/language/get

获取设备语言。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "language"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "language", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST language set

POST /system/language/set

设置设备语言。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "language",
            "value": 0
        },
        {
            "key": "language",
            "value": 1
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET language ability

GET /system/language/ability

获取 language 能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "language",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "English", "value": 0},
          {"label": "中文", "value": 1}
        ]
      }
    }
  ],
  "message": ""
}
```

## POST menu status get

POST /system/menu-status

获取菜单状态。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。

> Body 请求参数

```json
{
    "keys": [
        "status"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "status", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

## POST menu operator set

POST /system/menu-operator

菜单操作。仅支持单向操作，向一个方向操作时另一方向设为 0；进入菜单用 1，退出菜单用 -1。

> Body 请求参数

```json
{
    "panDir": 1,
    "tiltDir": 1
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "status": 0
  },
  "message": ""
}
```

## GET menu ability

GET /system/menu/ability

获取 menu 能力集（菜单状态 + 菜单操作）。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "status",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "panDir",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "Left (-1)", "value": -1},
          {"label": "停止 (0)", "value": 0},
          {"label": "Right (1)", "value": 1}
        ]
      }
    },
    {
      "key": "tiltDir",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "Down (-1)", "value": -1},
          {"label": "停止 (0)", "value": 0},
          {"label": "Up (1)", "value": 1}
        ]
      }
    }
  ],
  "message": ""
}
```

## POST osd display config get

POST /system/osd-display-config/get

获取 OSD 显示配置。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。bitmap 为位掩码：HDMI(0x01<<0) / SDI3G(0x01<<1) / PANORAMA_FULL(0x01<<2) / CLOSEUP_FULL(0x01<<3)。

> Body 请求参数

```json
{
    "keys": [
        "bitmap"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "bitmap", "value": 15, "enabled": true}
  ],
  "message": ""
}
```

## POST osd display config set

POST /system/osd-display-config/set

设置 OSD 显示配置。支持一次下发多组 key + value。bitmap 为位掩码：HDMI(0x01<<0) / SDI3G(0x01<<1) / PANORAMA_FULL(0x01<<2) / CLOSEUP_FULL(0x01<<3)。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "bitmap",
            "value": 15
        },
        {
            "key": "bitmap",
            "value": 3
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET osd display config ability

GET /system/osd-display-config/ability

获取 OSD 显示配置能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。bitmap 为位掩码：HDMI(0x01<<0) / SDI3G(0x01<<1) / PANORAMA_FULL(0x01<<2) / CLOSEUP_FULL(0x01<<3)。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "bitmap",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "min": 0,
        "max": 15,
        "step": 1
      }
    }
  ],
  "message": ""
}
```

## POST info get

POST /info/get

获取设备信息。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。修改后需调用 /info/apply 才会生效。

> Body 请求参数

```json
{
    "keys": [
        "deviceName",
        "product_name"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "deviceName", "value": "PBC-120-F", "enabled": true},
    {"key": "product_name", "value": "PBC-120-F", "enabled": true}
  ],
  "message": ""
}
```

## POST info set

POST /info/set

设置设备信息。支持一次下发多组 key + value。修改后需调用 /info/apply 才会生效。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "deviceName",
            "value": "PBC-120-F"
        },
        {
            "key": "product_name",
            "value": "PBC-120-F"
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET info ability

GET /info/ability

获取设备信息能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。修改通过 /info/set 下发，立即生效（无 apply 步骤）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "deviceName",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "PBC-120-F",
      "feature": {
        "maxLength": 64
      }
    },
    {
      "key": "product_name",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "PBC-120-F",
      "feature": {
        "maxLength": 64
      }
    }
  ],
  "message": ""
}
```

## POST time get

POST /time/get

获取设备时间和 NTP 参数。请求体带 keys 枚举数组，后端按 keys 返对应字段的当前值。修改后立即生效。

> Body 请求参数

```json
{
    "keys": [
        "timeSetting",
        "datetimeFormat",
        "manualDeviceTime",
        "ntpEnable",
        "ntpInterval",
        "ntpMainServer",
        "ntpStandbyServer",
        "timeZone"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "timeSetting", "value": 0, "enabled": true},
    {"key": "datetimeFormat", "value": 0, "enabled": true},
    {"key": "manualDeviceTime", "value": "2027-07-04T15:55:22Z", "enabled": true},
    {"key": "ntpEnable", "value": 1, "enabled": true},
    {"key": "ntpInterval", "value": 60, "enabled": true},
    {"key": "ntpMainServer", "value": "10.0.5.24", "enabled": true},
    {"key": "ntpStandbyServer", "value": "10.0.5.25", "enabled": true},
    {"key": "timeZone", "value": 32, "enabled": true}
  ],
  "message": ""
}
```

## POST time set

POST /time/set

设置设备时间和 NTP 参数。支持一次下发多组 key + value，立即生效。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "ntpMainServer",
            "value": "10.0.5.24"
        },
        {
            "key": "ntpStandbyServer",
            "value": "10.0.5.25"
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET time ability

GET /time/ability

获取设备时间能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。修改通过 /time/set 下发，立即生效（无 apply 步骤）。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "timeSetting",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "手动", "value": 0},
          {"label": "NTP 同步", "value": 1}
        ]
      }
    },
    {
      "key": "datetimeFormat",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0,
      "feature": {
        "options": [
          {"label": "YYYY-MM-DD", "value": 0},
          {"label": "MM/DD/YYYY", "value": 1},
          {"label": "DD/MM/YYYY", "value": 2}
        ]
      }
    },
    {
      "key": "manualDeviceTime",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "2027-07-04T15:55:22Z",
      "feature": {
        "pattern": "ISO 8601 datetime"
      }
    },
    {
      "key": "ntpEnable",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "ntpInterval",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 60,
      "feature": {
        "min": 1,
        "max": 1440,
        "step": 1
      }
    },
    {
      "key": "ntpMainServer",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "10.0.5.24",
      "feature": {
        "pattern": "IPv4"
      }
    },
    {
      "key": "ntpStandbyServer",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "10.0.5.25",
      "feature": {
        "pattern": "IPv4"
      }
    },
    {
      "key": "timeZone",
      "component": "Select",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 32,
      "feature": {
        "options": [
          {"label": "UTC", "value": 0},
          {"label": "GMT+8 北京", "value": 32}
        ]
      }
    }
  ],
  "message": ""
}
```

## POST ethernet get

POST /ethernet/get

获取网卡参数。请求体带 id（网卡 ID）和 keys 枚举数组，后端按 keys 返对应字段的当前值。修改后需调用 /ethernet/apply 才会生效。

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "ipaddr",
        "netmask",
        "gateway",
        "dns1",
        "dns2",
        "isDhcp",
        "isAutoDns",
        "isEnabled"
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "ipaddr", "value": "10.0.6.226", "enabled": true},
    {"key": "netmask", "value": "255.255.255.0", "enabled": true},
    {"key": "gateway", "value": "10.0.6.1", "enabled": true},
    {"key": "dns1", "value": "10.0.6.1", "enabled": true},
    {"key": "dns2", "value": "0.0.0.0", "enabled": true},
    {"key": "isDhcp", "value": 0, "enabled": true},
    {"key": "isAutoDns", "value": 0, "enabled": true},
    {"key": "isEnabled", "value": 1, "enabled": true}
  ],
  "message": ""
}
```

## POST ethernet set

POST /ethernet/set

设置网卡参数。支持一次下发多组 key + value。修改后需调用 /ethernet/apply 才会生效。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "ipaddr",
            "value": "10.0.6.226"
        },
        {
            "key": "netmask",
            "value": "255.255.255.0"
        }
    ]
}
```

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "ok",
  "message": ""
}
```

## GET ethernet ability

GET /ethernet/ability

获取网卡能力集。data 字段为各可配置项的 component/span/order/show/default/feature（options 列表）。修改通过 /ethernet/set 下发，立即生效（无 apply 步骤）。id 标识哪一张网卡。

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "key": "ipaddr",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "10.0.6.226",
      "feature": {
        "pattern": "^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$"
      }
    },
    {
      "key": "netmask",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "255.255.255.0",
      "feature": {
        "pattern": "IPv4 netmask"
      }
    },
    {
      "key": "gateway",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "10.0.6.1",
      "feature": {
        "pattern": "IPv4"
      }
    },
    {
      "key": "dns1",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "10.0.6.1",
      "feature": {
        "pattern": "IPv4"
      }
    },
    {
      "key": "dns2",
      "component": "Input",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": "0.0.0.0",
      "feature": {
        "pattern": "IPv4"
      }
    },
    {
      "key": "isDhcp",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "isAutoDns",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 0
    },
    {
      "key": "isEnabled",
      "component": "Switch",
      "span": 8,
      "order": -1,
      "show": 1,
      "default": 1
    }
  ],
  "message": ""
}
```

# PBC-120/ISP

## GET isp ability

GET /isp/ability

获取 ISP 能力集

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": [
    {
      "component": "string",
      "default": 0,
      "key": "string",
      "order": 0,
      "show": 0,
      "span": 0,
      "feature": {
        "input": 0,
        "max": 0,
        "min": 0,
        "step": 0
      }
    }
  ],
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST isp get

POST /isp/get

> Body 请求参数

```json
{
    "id": 0,
    "keys": [
        "flip_x",
        "img_2d_noise",
        "contrast",
        "sharpness",
        "row_end_1",
        "flip_y",
        "img_3d_noise",
        "gamma",
        "brightness",
        "drc"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[ISP Key](#schemaisp key)] | 是   |     | Function key collection |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | imgStyle |
| » keys | contrast |
| » keys | sharpness |
| » keys | saturation |
| » keys | gamma |
| » keys | brightness |
| » keys | hdrSwi |
| » keys | expMode |
| » keys | antiFlicker |
| » keys | backlightComp |
| » keys | shutterSpeed |
| » keys | gain |
| » keys | gainLimit |
| » keys | expBrightness |
| » keys | expCompSwi |
| » keys | expComp |
| » keys | wbMode |
| » keys | hue |
| » keys | wbSensitivity |
| » keys | wbRstrength |
| » keys | wbGstrength |
| » keys | wbBstrength |
| » keys | wbRed |
| » keys | wbBlue |
| » keys | wbGreen |
| » keys | wbTemperature |
| » keys | wbOneKey |
| » keys | mirror |
| » keys | filp |
| » keys | colorHueBlue |
| » keys | colorHueCyan |
| » keys | colorHueGreen |
| » keys | colorHueMagenta |
| » keys | colorHueRed |
| » keys | colorHueYellow |
| » keys | colorSaturationBlue |
| » keys | colorSaturationCyan |
| » keys | colorSaturationGreen |
| » keys | colorSaturationMagenta |
| » keys | colorSaturationRed |
| » keys | colorSaturationYellow |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "ae", "value": 0, "enabled": true},
    {"key": "brightness", "value": 50, "enabled": true},
    {"key": "contrast", "value": 50, "enabled": true}
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST isp set

POST /isp/set

支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "id": 0,
    "data": [
        {
            "key": "sharpness",
            "value": 27
        },
        {
            "key": "contrast",
            "value": 50
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | imgStyle |
| » key | contrast |
| » key | sharpness |
| » key | saturation |
| » key | gamma |
| » key | brightness |
| » key | hdrSwi |
| » key | expMode |
| » key | antiFlicker |
| » key | backlightComp |
| » key | shutterSpeed |
| » key | gain |
| » key | gainLimit |
| » key | expBrightness |
| » key | expCompSwi |
| » key | expComp |
| » key | wbMode |
| » key | hue |
| » key | wbSensitivity |
| » key | wbRstrength |
| » key | wbGstrength |
| » key | wbBstrength |
| » key | wbRed |
| » key | wbBlue |
| » key | wbGreen |
| » key | wbTemperature |
| » key | wbOneKey |
| » key | mirror |
| » key | filp |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## GET custom isp style

GET /custom-isp-style/get

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": {
    "cfgs_alias": "会议室",
    "cfgs_id": 1,
    "cfgs_size": 10,
    "current_cfg": 1
  },
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | object | true | none |     | none |
| »» cfgs_alias | [string] | true | none |     | none |
| »» cfgs_id | [integer] | true | none |     | none |
| »» cfgs_size | integer | true | none |     | none |
| »» current_cfg | integer | true | none |     | none |
| » message | string | true | none |     | none |

## POST custom isp style

POST /custom-isp-style/set

> Body 请求参数

```json
{
    "id": 0,
    "method": 0,
    "cfg_alias": ""
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » id | body | integer | 是   |     | none |
| » method | body | string | 是   |     | none |
| » cfg_alias | body | string |     |     | method为alias时，cfg_alias不能为空 |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » method | set |
| » method | recall |
| » method | delete |
| » method | alias |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": "success",
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

# PBC-120/PT

## GET pt ability

GET /pt/ability

获取 PT 能力集

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": [
    {
      "component": "string",
      "default": 0,
      "key": "string",
      "order": 0,
      "show": 0,
      "span": 0,
      "feature": {
        "input": 0,
        "max": 0,
        "min": 0,
        "step": 0
      }
    }
  ],
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST pt get

POST /pt/get

> Body 请求参数

```json
{
    "keys": [
        "zoomSpeed",
        "ptPanSpeed",
        "ptTiltSpeed",
        "power_up_pos",
        "preset_speed",
        "speed_match",
        "digital_zoom",
        "preset_freeze"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[PT Key](#schemapt key)] | 是   |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | ptPanSpeed |
| » keys | ptTiltSpeed |
| » keys | zoomSpeed |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST pt set

POST /pt/set

支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "data": [
        {
            "key": "ptPanSpeed",
            "value": 4
        },
        {
            "key": "ptTiltSpeed",
            "value": 4
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | ptPanSpeed |
| » key | ptTiltSpeed |
| » key | zoomSpeed |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST pt move

POST /pt/rel-move

PTZ movement

> Body 请求参数

```json
{
    "panDir": 0,
    "tiltDir": 0
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » panDir | body | integer | 是   |     | Horizontal move direction |
| » tiltDir | body | integer | 是   |     | Vertical move direction |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » panDir | -1  |
| » panDir | 0   |
| » panDir | 1   |
| » tiltDir | -1  |
| » tiltDir | 0   |
| » tiltDir | 1   |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST pt point

POST /pt/point

Preset operation

> Body 请求参数

```json
{
    "method": "set",
    "id": 1
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » method | body | string | 是   |     | Preset operation method |
| » id | body | integer | 是   |     | Preset ID |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » method | recall |
| » method | set |
| » method | delete |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": "string",
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | string | true | none |     | none |
| » message | string | true | none |     | none |

# PBC-120/AF

## GET af ability

GET /af/ability

获取 AF 能力集

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": [
    {
      "component": "string",
      "default": 0,
      "key": "string",
      "order": 0,
      "show": 0,
      "span": 0,
      "feature": {
        "input": 0,
        "max": 0,
        "min": 0,
        "step": 0
      }
    }
  ],
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST af get

POST /af/get

> Body 请求参数

```json
{
    "keys": [
        "af_speed",
        "af_auto",
        "af_area",
        "af_mode",
        "af_face",
        "af_ptz_help"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST af set

POST /af/set

支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "data": [
        {
            "key": "af_speed",
            "value": 4
        },
        {
            "key": "af_auto",
            "value": 0
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

# PBC-120/Track

## GET track ability

GET /track/ability

获取跟踪能力集

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| panel | query | string | 否   |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| panel | track |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": {},
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST basic get

POST /track/basic/get

Track Param Get

> Body 请求参数

```json
{
    "keys": [
        "trackMode",
        "gestureTrigger",
        "designatedId",
        "takeTurn"
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » keys | body | [[Track Key](#schematrack key)] | 是   |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » keys | lostAction |
| » keys | lostTime |
| » keys | sensitivity |
| » keys | trackSpeed |
| » keys | headPos |
| » keys | bodySize |
| » keys | audioLostTime |
| » keys | audioLossPos |
| » keys | trackMode |
| » keys | gestureTrigger |
| » keys | takeTurn |
| » keys | designatedId |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {"key": "trackMode", "value": 0, "enabled": true},
    {"key": "gestureTrigger", "value": 0, "enabled": true},
    {"key": "takeTurn", "value": 0, "enabled": true},
    {"key": "designatedId", "value": 0, "enabled": true}
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [object] | true | none |     | none |
| »» enabled | boolean | true | none |     | none |
| »» key | string | true | none |     | none |
| »» value | integer | true | none |     | none |
| » message | string | true | none |     | none |

## POST basic set

POST /track/basic/set

Track Param Set。支持一次下发多组 key + value。

> Body 请求参数

```json
{
    "data": [
        {
            "key": "gestureTrigger",
            "value": 0
        },
        {
            "key": "trackMode",
            "value": 0
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » data | body | [[PanelSetValue](#schemapanelsetvalue)] | 是   |     | 多组 key + value |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » key | lostAction |
| » key | lostTime |
| » key | sensitivity |
| » key | trackSpeed |
| » key | headPos |
| » key | bodySize |
| » key | audioLostTime |
| » key | audioLossPos |
| » key | trackMode |
| » key | gestureTrigger |
| » key | takeTurn |
| » key | designatedId |

> 返回示例

> 200 Response

```json
{
  "code": 0,
  "data": [
    {
      "enabled": true,
      "key": "string",
      "value": 0,
      "visible": true
    }
  ],
  "message": "string"
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [object] | true | none |     | none |
| »» enabled | boolean | false | none |     | none |
| »» key | string | false | none |     | none |
| »» value | integer | false | none |     | none |
| »» visible | boolean | false | none |     | none |
| » message | string | true | none |     | none |

## GET track area get

GET /track/area

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| id  | query | [TrackArea](#schematrackarea) | 否   |     | Track Area ID |
| trackMode | query | [TrackMode](#schematrackmode) | 否   |     | Track Mode |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| id  | PLATFORM |
| id  | SHIELD |
| id  | PRIORITY |
| id  | PRESET |
| id  | BLACKBOARD |
| trackMode | 15  |
| trackMode | 0   |
| trackMode | 1   |
| trackMode | 2   |
| trackMode | 3   |

> 返回示例

> 200 Response

```json
{
  "code": 200,
  "data": [
    {
      "LeftDownX": 0,
      "LeftDownY": 0,
      "LeftTopX": 0,
      "LeftTopY": 0,
      "RightDownX": 0,
      "RightDownY": 0,
      "RightTopX": 0,
      "RightTopY": 0,
      "enable": false,
      "presetId": 0,
      "workMode": 0
    }
  ],
  "message": ""
}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

状态码 **200**

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| » code | integer | true | none |     | none |
| » data | [[TrackArea1](#schematrackarea1)] | true | none |     | none |
| »» LeftDownX | integer | false | none | Left Bottom X | X coordinate of left bottom corner of track area, range 0-3840 |
| »» LeftDownY | integer | false | none | Left Bottom Y | Y coordinate of left bottom corner of track area, range 0-2160 |
| »» LeftTopX | integer | false | none | Left Top X | X coordinate of left top corner of track area, range 0-3840 |
| »» LeftTopY | integer | false | none | Left Top Y | Y coordinate of left top corner of track area, range 0-2160 |
| »» RightDownX | integer | false | none | Right Bottom X | X coordinate of right bottom corner of track area, range 0-3840 |
| »» RightDownY | integer | false | none | Right Bottom Y | Y coordinate of right bottom corner of track area, range 0-2160 |
| »» RightTopX | integer | false | none | Right Top X | X coordinate of right top corner of track area, range 0-3840 |
| »» RightTopY | integer | false | none | Right Top Y | Y coordinate of right top corner of track area, range 0-2160 |
| »» enable | boolean | false | none | Enable Status | Whether this track area is enabled |
| »» presetId | integer | false | none | Preset ID | Preset bound to this track area |
| »» workMode | integer | false | none | Work Mode | Currently selected work mode |
| » message | string | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| workMode | 0   |
| workMode | 1   |
| workMode | 2   |

## POST track area set

POST /track/area

> Body 请求参数

```json
{
    "trackMode": 3,
    "id": "PRIORITY",
    "list": [
        {
            "LeftTopX": 0,
            "LeftDownX": 0,
            "LeftTopY": 0,
            "LeftDownY": 0,
            "RightTopX": 0,
            "RightDownX": 0,
            "RightTopY": 0,
            "RightDownY": 0,
            "presetId": 0,
            "enable": false,
            "workMode": 0
        }
    ]
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » trackMode | body | [TrackMode](#schematrackmode) | 是   |     | Track Mode |
| » id | body | [TrackArea](#schematrackarea) | 是   |     | Track Area ID |
| » list | body | [[TrackArea1](#schematrackarea1)] | 是   |     | none |
| »» LeftDownX | body | integer | 否   | Left Bottom X | X coordinate of left bottom corner of track area, range 0-3840 |
| »» LeftDownY | body | integer | 否   | Left Bottom Y | Y coordinate of left bottom corner of track area, range 0-2160 |
| »» LeftTopX | body | integer | 否   | Left Top X | X coordinate of left top corner of track area, range 0-3840 |
| »» LeftTopY | body | integer | 否   | Left Top Y | Y coordinate of left top corner of track area, range 0-2160 |
| »» RightDownX | body | integer | 否   | Right Bottom X | X coordinate of right bottom corner of track area, range 0-3840 |
| »» RightDownY | body | integer | 否   | Right Bottom Y | Y coordinate of right bottom corner of track area, range 0-2160 |
| »» RightTopX | body | integer | 否   | Right Top X | X coordinate of right top corner of track area, range 0-3840 |
| »» RightTopY | body | integer | 否   | Right Top Y | Y coordinate of right top corner of track area, range 0-2160 |
| »» enable | body | boolean | 否   | Enable Status | Whether this track area is enabled |
| »» presetId | body | integer | 否   | Preset ID | Preset bound to this track area |
| »» workMode | body | integer | 否   | Work Mode | Currently selected work mode |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| » trackMode | 15  |
| » trackMode | 0   |
| » trackMode | 1   |
| » trackMode | 2   |
| » trackMode | 3   |
| » id | PLATFORM |
| » id | SHIELD |
| » id | PRIORITY |
| » id | PRESET |
| » id | BLACKBOARD |
| »» workMode | 0   |
| »» workMode | 1   |
| »» workMode | 2   |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

## POST designated trigger

POST /track/designated-trigger

> Body 请求参数

```json
{
    "x": 0,
    "y": 0
}
```

### 请求参数

| 名称  | 位置  | 类型  | 必选  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| body | body | object | 是   |     | none |
| » x | body | integer | 是   |     | 0-1之间的三位小数，对应视频水平方向上的点的比例 |
| » y | body | integer | 是   |     | 0-1之间的三位小数，对应视频水垂直方向上的点的比例 |

> 返回示例

> 200 Response

```json
{}
```

### 返回结果

| 状态码 | 状态码含义 | 说明  | 数据模型 |
| --- | --- | --- | --- |
| 200 | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | none | Inline |

### 返回数据结构

<h2 id="tocS_Info Key">Info Key</h2>

<a id="schemainfo key"></a>
<a id="schema_Info Key"></a>
<a id="tocSinfo key"></a>
<a id="tocsinfo key"></a>

```json
"deviceName"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | deviceName |
| *anonymous* | product_name |

<h2 id="tocS_Time Key">Time Key</h2>

<a id="schematime key"></a>
<a id="schema_Time Key"></a>
<a id="tocStime key"></a>
<a id="tocstime key"></a>

```json
"timeSetting"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | timeSetting |
| *anonymous* | datetimeFormat |
| *anonymous* | manualDeviceTime |
| *anonymous* | ntpEnable |
| *anonymous* | ntpInterval |
| *anonymous* | ntpMainServer |
| *anonymous* | ntpStandbyServer |
| *anonymous* | timeZone |

<h2 id="tocS_Ethernet Key">Ethernet Key</h2>

<a id="schemaethernet key"></a>
<a id="schema_Ethernet Key"></a>
<a id="tocSethernet key"></a>
<a id="tocsethernet key"></a>

```json
"ipaddr"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | ipaddr |
| *anonymous* | netmask |
| *anonymous* | gateway |
| *anonymous* | dns1 |
| *anonymous* | dns2 |
| *anonymous* | isDhcp |
| *anonymous* | isAutoDns |
| *anonymous* | isEnabled |

# 数据模型

<h2 id="tocS_Lens Channel">Lens Channel</h2>

<a id="schemalens channel"></a>
<a id="schema_Lens Channel"></a>
<a id="tocSlens channel"></a>
<a id="tocslens channel"></a>

```json
0
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | integer | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | 0   |
| *anonymous* | 1   |

<h2 id="tocS_PT Key">PT Key</h2>

<a id="schemapt key"></a>
<a id="schema_PT Key"></a>
<a id="tocSpt key"></a>
<a id="tocspt key"></a>

```json
"ptPanSpeed"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | ptPanSpeed |
| *anonymous* | ptTiltSpeed |
| *anonymous* | zoomSpeed |

<h2 id="tocS_Port">Port</h2>

<a id="schemaport"></a>
<a id="schema_Port"></a>
<a id="tocSport"></a>
<a id="tocsport"></a>

```json
1
```

Port

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| Port | integer | false | none | Port | none |

<h2 id="tocS_TrackMode">TrackMode</h2>

<a id="schematrackmode"></a>
<a id="schema_TrackMode"></a>
<a id="tocStrackmode"></a>
<a id="tocstrackmode"></a>

```json
15
```

Track Mode

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | integer | false | none |     | Track Mode |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | 15  |
| *anonymous* | 0   |
| *anonymous* | 1   |
| *anonymous* | 2   |
| *anonymous* | 3   |

<h2 id="tocS_TrackArea">TrackArea</h2>

<a id="schematrackarea"></a>
<a id="schema_TrackArea"></a>
<a id="tocStrackarea"></a>
<a id="tocstrackarea"></a>

```json
"PLATFORM"
```

Track Area

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | Track Area |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | PLATFORM |
| *anonymous* | SHIELD |
| *anonymous* | PRIORITY |
| *anonymous* | PRESET |
| *anonymous* | BLACKBOARD |

<h2 id="tocS_ISP Key">ISP Key</h2>

<a id="schemaisp key"></a>
<a id="schema_ISP Key"></a>
<a id="tocSisp key"></a>
<a id="tocsisp key"></a>

```json
"imgStyle"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | imgStyle |
| *anonymous* | contrast |
| *anonymous* | sharpness |
| *anonymous* | saturation |
| *anonymous* | gamma |
| *anonymous* | brightness |
| *anonymous* | hdrSwi |
| *anonymous* | expMode |
| *anonymous* | antiFlicker |
| *anonymous* | backlightComp |
| *anonymous* | shutterSpeed |
| *anonymous* | gain |
| *anonymous* | gainLimit |
| *anonymous* | expBrightness |
| *anonymous* | expCompSwi |
| *anonymous* | expComp |
| *anonymous* | wbMode |
| *anonymous* | hue |
| *anonymous* | wbSensitivity |
| *anonymous* | wbRstrength |
| *anonymous* | wbGstrength |
| *anonymous* | wbBstrength |
| *anonymous* | wbRed |
| *anonymous* | wbBlue |
| *anonymous* | wbGreen |
| *anonymous* | wbTemperature |
| *anonymous* | wbOneKey |
| *anonymous* | mirror |
| *anonymous* | filp |

<h2 id="tocS_Enable">Enable</h2>

<a id="schemaenable"></a>
<a id="schema_Enable"></a>
<a id="tocSenable"></a>
<a id="tocsenable"></a>

```json
0
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | integer | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | 0   |
| *anonymous* | 1   |

<h2 id="tocS_Stream Index">Stream Index</h2>

<a id="schemastream index"></a>
<a id="schema_Stream Index"></a>
<a id="tocSstream index"></a>
<a id="tocsstream index"></a>

```json
1
```

Stream Type

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| Stream Type | integer | false | none | Stream Type | 1->Main stream |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| Stream Type | 1   |

<h2 id="tocS_Upgrade Protect">Upgrade Protect</h2>

<a id="schemaupgrade protect"></a>
<a id="schema_Upgrade Protect"></a>
<a id="tocSupgrade protect"></a>
<a id="tocsupgrade protect"></a>

```json
1
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | integer | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | 1   |
| *anonymous* | 4   |
| *anonymous* | 8   |
| *anonymous* | 16  |
| *anonymous* | 64  |
| *anonymous* | 128 |
| *anonymous* | 256 |
| *anonymous* | 512 |
| *anonymous* | 1024 |
| *anonymous* | 2048 |
| *anonymous* | 4096 |
| *anonymous* | 8192 |
| *anonymous* | 16384 |
| *anonymous* | 32768 |
| *anonymous* | 65536 |
| *anonymous* | 131072 |
| *anonymous* | 262144 |
| *anonymous* | 524288 |
| *anonymous* | 1048576 |
| *anonymous* | 2097152 |
| *anonymous* | 4194304 |
| *anonymous* | 8388608 |
| *anonymous* | 16777216 |
| *anonymous* | 33554432 |

<h2 id="tocS_Track Key">Track Key</h2>

<a id="schematrack key"></a>
<a id="schema_Track Key"></a>
<a id="tocStrack key"></a>
<a id="tocstrack key"></a>

```json
"lostAction"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | lostAction |
| *anonymous* | lostTime |
| *anonymous* | sensitivity |
| *anonymous* | trackSpeed |
| *anonymous* | headPos |
| *anonymous* | bodySize |
| *anonymous* | audioLostTime |
| *anonymous* | audioLossPos |
| *anonymous* | trackMode |
| *anonymous* | gestureTrigger |
| *anonymous* | takeTurn |
| *anonymous* | designatedId |

<h2 id="tocS_Visca Passthrough">Visca Passthrough</h2>

<a id="schemavisca passthrough"></a>
<a id="schema_Visca Passthrough"></a>
<a id="tocSvisca passthrough"></a>
<a id="tocsvisca passthrough"></a>

```json
{
  "address": "192.168.0.1",
  "enable": true,
  "port": 1,
  "protocolType": true,
  "viscaAddr": 1,
  "workMode": true
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| address | string(ipv4) | true | none | Communication Address | none |
| enable | boolean | true | none | Enable | none |
| port | integer | true | none | Port | none |
| protocolType | boolean | true | none | Communication Protocol | true->TCP, false->UDP |
| viscaAddr | integer | true | none | Visca Address | none |
| workMode | boolean | true | none | Communication Mode | true->Server, false->Client |

<h2 id="tocS_Input Serial Port">Input Serial Port</h2>

<a id="schemainput serial port"></a>
<a id="schema_Input Serial Port"></a>
<a id="tocSinput serial port"></a>
<a id="tocsinput serial port"></a>

```json
{
  "BaudrateIn": 0,
  "ComModeIn": 0,
  "ProtocolTypeIn": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| BaudrateIn | integer | true | none |     | none |
| ComModeIn | integer | true | none |     | none |
| ProtocolTypeIn | integer | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| BaudrateIn | 0   |
| BaudrateIn | 1   |
| BaudrateIn | 2   |
| BaudrateIn | 3   |
| ComModeIn | 0   |
| ComModeIn | 1   |
| ProtocolTypeIn | 0   |
| ProtocolTypeIn | 1   |
| ProtocolTypeIn | 2   |
| ProtocolTypeIn | 3   |

<h2 id="tocS_IR Control">IR Control</h2>

<a id="schemair control"></a>
<a id="schema_IR Control"></a>
<a id="tocSir control"></a>
<a id="tocsir control"></a>

```json
{
  "Ir_addr": 1,
  "bEnable": true
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| Ir_addr | integer | true | none |     | none |
| bEnable | boolean | true | none |     | none |

<h2 id="tocS_PelcoPD Control">PelcoPD Control</h2>

<a id="schemapelcopd control"></a>
<a id="schema_PelcoPD Control"></a>
<a id="tocSpelcopd control"></a>
<a id="tocspelcopd control"></a>

```json
{
  "PelcoD_addr": 255,
  "PelcoP_addr": 31
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| PelcoD_addr | integer | true | none |     | none |
| PelcoP_addr | integer | true | none |     | none |

<h2 id="tocS_USB Control">USB Control</h2>

<a id="schemausb control"></a>
<a id="schema_USB Control"></a>
<a id="tocSusb control"></a>
<a id="tocsusb control"></a>

```json
{
  "usbDevice": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| usbDevice | integer | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| usbDevice | 0   |
| usbDevice | 1   |

<h2 id="tocS_Tally Control">Tally Control</h2>

<a id="schematally control"></a>
<a id="schema_Tally Control"></a>
<a id="tocStally control"></a>
<a id="tocstally control"></a>

```json
{
  "customRgb": 16777216,
  "ndiTally": 0,
  "tallyResetStatus": 0,
  "trackTally": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| customRgb | integer | true | none |     | 十六进制RGB颜色值转为int的值，#000000-#FFFF00，不支持蓝色 |
| ndiTally | [Enable](#schemaenable) | true | none |     | none |
| tallyResetStatus | integer | true | none |     | none |
| trackTally | [Enable](#schemaenable) | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| tallyResetStatus | 0   |
| tallyResetStatus | 1   |

<h2 id="tocS_KXWELL Control">KXWELL Control</h2>

<a id="schemakxwell control"></a>
<a id="schema_KXWELL Control"></a>
<a id="tocSkxwell control"></a>
<a id="tocskxwell control"></a>

```json
{
  "kxwell_addr": 1
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| kxwell_addr | integer | true | none |     | none |

<h2 id="tocS_Visca透传">Visca透传</h2>

<a id="schemavisca透传"></a>
<a id="schema_Visca透传"></a>
<a id="tocSvisca透传"></a>
<a id="tocsvisca透传"></a>

```json
{
  "address": "192.168.0.1",
  "enable": true,
  "port": 1,
  "protocolType": true,
  "viscaAddr": 1,
  "workMode": true
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| address | string(ipv4) | true | none | 通信地址 | none |
| enable | boolean | true | none | 启用  | none |
| port | integer | true | none | 端口  | none |
| protocolType | boolean | true | none | 通信协议 | true->TCP, false->UDP |
| viscaAddr | integer | true | none | Visca 地址 | none |
| workMode | boolean | true | none | 通信模式 | true->服务端, false->客户端 |

<h2 id="tocS_Visca Over IP">Visca Over IP</h2>

<a id="schemavisca over ip"></a>
<a id="schema_Visca Over IP"></a>
<a id="tocSvisca over ip"></a>
<a id="tocsvisca over ip"></a>

```json
{
  "enable": true,
  "port": 1
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| enable | boolean | true | none | Enable | none |
| port | integer | true | none | Port | none |

<h2 id="tocS_FreeD参数">FreeD参数</h2>

<a id="schemafreed参数"></a>
<a id="schema_FreeD参数"></a>
<a id="tocSfreed参数"></a>
<a id="tocsfreed参数"></a>

```json
{
  "caremaId": 1,
  "enable": 0,
  "ipAddr": "192.168.0.1",
  "port": 1
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| caremaId | integer | true | none |     | 摄像头ID，1-255 |
| enable | [Enable](#schemaenable) | true | none |     | 开关  |
| ipAddr | string(ipv4) | true | none |     | IP地址 |
| port | integer | true | none |     | 端口  |

<h2 id="tocS_MDSN参数">MDSN参数</h2>

<a id="schemamdsn参数"></a>
<a id="schema_MDSN参数"></a>
<a id="tocSmdsn参数"></a>
<a id="tocsmdsn参数"></a>

```json
{
  "hostname": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| hostname | string | true | none |     | 主机名 |

<h2 id="tocS_Video Key">Video Key</h2>

<a id="schemavideo key"></a>
<a id="schema_Video Key"></a>
<a id="tocSvideo key"></a>
<a id="tocsvideo key"></a>

```json
"codec"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | BitrateProfile |
| *anonymous* | QuantFactorI |
| *anonymous* | QuantFactorP |
| *anonymous* | bitrate |
| *anonymous* | codec |
| *anonymous* | fps |
| *anonymous* | gop |
| *anonymous* | profile |
| *anonymous* | rc  |
| *anonymous* | size |

<h2 id="tocS_Video Param">Video Param</h2>

<a id="schemavideo param"></a>
<a id="schema_Video Param"></a>
<a id="tocSvideo param"></a>
<a id="tocsvideo param"></a>

```json
{
  "BitrateProfile": 0,
  "QuantFactorI": 14,
  "QuantFactorP": 17,
  "bitrate": 1000,
  "codec": 1,
  "fps": 15,
  "gop": 1,
  "id": "video_main_close_up",
  "profile": 1,
  "rc": 0,
  "size": 1
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| BitrateProfile | integer | true | none | Bitrate Profile | none |
| QuantFactorI | integer | true | none | Quant Factor I-Frame | none |
| QuantFactorP | integer | true | none | Quant Factor P-Frame | none |
| bitrate | integer | true | none | Bitrate | none |
| codec | integer | true | none | Codec Type | none |
| fps | integer | true | none | Frame Rate | none |
| gop | integer | true | none | Key Frame Interval | none |
| id  | string | true | none | Video Channel ID | none |
| profile | integer | true | none | Profile | none |
| rc  | integer | true | none | Rate Control | none |
| size | integer | true | none | Resolution | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| BitrateProfile | 0   |
| BitrateProfile | 1   |
| BitrateProfile | 2   |
| BitrateProfile | 3   |
| BitrateProfile | 4   |
| codec | 1   |
| codec | 2   |
| fps | 15  |
| fps | 30  |
| fps | 60  |
| id  | video_main_close_up |
| id  | video_main_panorama |
| profile | 1   |
| profile | 2   |
| rc  | 0   |
| rc  | 1   |
| size | 1   |
| size | 2   |
| size | 3   |
| size | 6   |

<h2 id="tocS_Audio Key">Audio Key</h2>

<a id="schemaaudio key"></a>
<a id="schema_Audio Key"></a>
<a id="tocSaudio key"></a>
<a id="tocsaudio key"></a>

```json
"codec"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | audioInput |
| *anonymous* | bitrate |
| *anonymous* | channels |
| *anonymous* | codec |
| *anonymous* | mute |
| *anonymous* | samplerate |
| *anonymous* | volume |

<h2 id="tocS_Audio Param">Audio Param</h2>

<a id="schemaaudio param"></a>
<a id="schema_Audio Param"></a>
<a id="tocSaudio param"></a>
<a id="tocsaudio param"></a>

```json
{
  "audioInput": 1,
  "bitrate": 48000,
  "channels": 1,
  "codec": 1,
  "mute": true,
  "samplerate": 16000,
  "volume": 100
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| audioInput | integer | true | none |     | Input Source |
| bitrate | integer | true | none |     | Bitrate |
| channels | integer | true | none |     | Audio Channel |
| codec | integer | true | none |     | Codec |
| mute | boolean | true | none |     | Mute |
| samplerate | integer | true | none |     | Sample Rate |
| volume | integer | true | none |     | Volume |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| audioInput | 1   |
| audioInput | 2   |
| audioInput | 0   |
| bitrate | 48000 |
| bitrate | 64000 |
| bitrate | 96000 |
| bitrate | 128000 |
| channels | 1   |
| channels | 2   |
| codec | 1   |
| samplerate | 16000 |
| samplerate | 32000 |
| samplerate | 48000 |

<h2 id="tocS_RTSP Key">RTSP Key</h2>

<a id="schemartsp key"></a>
<a id="schema_RTSP Key"></a>
<a id="tocSrtsp key"></a>
<a id="tocsrtsp key"></a>

```json
{
    "audio": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | audio |
| *anonymous* | enable |
| *anonymous* | enableFixed |
| *anonymous* | streamName |
| *anonymous* | port |
| *anonymous* | rtspOverHttpEnable |
| *anonymous* | rtspOverHttpPort |
| *anonymous* | authEnable |
| *anonymous* | username |
| *anonymous* | password |

<h2 id="tocS_RTSP">RTSP</h2>

<a id="schemartsp"></a>
<a id="schema_RTSP"></a>
<a id="tocSrtsp"></a>
<a id="tocsrtsp"></a>

```json
{
  "audio": -1,
  "enable": true,
  "enableFixed": true,
  "index": 0,
  "streamName": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| audio | integer | true | none | Audio Output | Currently only closed. -1->Close, 6->AAC |
| enable | boolean | true | none | Enable | none |
| enableFixed | boolean | true | none |     | When false, enable cannot be set. Currently only true. |
| index | integer | true | none | RTSP Channel | Consistent with the parameter id when getting |
| streamName | string | true | none | Stream Name | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| audio | -1  |
| audio | 6   |
| index | 0   |

<h2 id="tocS_RTMP Key">RTMP Key</h2>

<a id="schemartmp key"></a>
<a id="schema_RTMP Key"></a>
<a id="tocSrtmp key"></a>
<a id="tocsrtmp key"></a>

```json
"video"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | audio |
| *anonymous* | authEnable |
| *anonymous* | enable |
| *anonymous* | enableFixed |
| *anonymous* | key |
| *anonymous* | password |
| *anonymous* | url |
| *anonymous* | username |
| *anonymous* | video |

<h2 id="tocS_RTMP">RTMP</h2>

<a id="schemartmp"></a>
<a id="schema_RTMP"></a>
<a id="tocSrtmp"></a>
<a id="tocsrtmp"></a>

```json
{
  "index": 0,
  "enable": true,
  "enableFixed": true,
  "video": 1,
  "audio": -1,
  "url": "string",
  "key": "string",
  "status": "unknown",
  "authEnable": true,
  "username": "string",
  "password": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| index | integer | true | none | RTMP Channel | Consistent with the parameter id when getting |
| enable | boolean | true | none | Enable | none |
| enableFixed | boolean | true | read-only |     | When false, enable cannot be set. Currently only true. |
| video | [Stream Index](#schemastream index) | true | none | Video Output | 1->Main stream |
| audio | integer | true | none | Audio Output | Currently only closed. -1->Close, 6->AAC |
| url | string | true | none | Server URL | none |
| key | string | true | none | Server Key | none |
| status | string | true | read-only | Connection Status | 状态  |
| authEnable | boolean | true | none | Auth Switch | none |
| username | string | true | none | Username | none |
| password | string | true | none | Password | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| index | 0   |
| audio | -1  |
| audio | 6   |
| status | unknown |
| status | ok  |
| status | bad |

<h2 id="tocS_SRT Key">SRT Key</h2>

<a id="schemasrt key"></a>
<a id="schema_SRT Key"></a>
<a id="tocSsrt key"></a>
<a id="tocssrt key"></a>

```json
"enable"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | aesEnable |
| *anonymous* | aesMode |
| *anonymous* | audio |
| *anonymous* | enable |
| *anonymous* | enableFixed |
| *anonymous* | hostname |
| *anonymous* | latency |
| *anonymous* | mode |
| *anonymous* | password |
| *anonymous* | port |
| *anonymous* | streamid |

<h2 id="tocS_SRT">SRT</h2>

<a id="schemasrt"></a>
<a id="schema_SRT"></a>
<a id="tocSsrt"></a>
<a id="tocssrt"></a>

```json
{
  "aesEnable": true,
  "aesMode": 16,
  "audio": -1,
  "enable": true,
  "enableFixed": true,
  "hostname": "string",
  "index": 0,
  "latency": 0,
  "mode": 1,
  "password": "string",
  "port": 1,
  "status": "unknown",
  "streamid": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| aesEnable | boolean | true | none |     | none |
| aesMode | integer | true | none |     | none |
| audio | integer | true | none |     | none |
| enable | boolean | true | none |     | none |
| enableFixed | boolean | true | none |     | none |
| hostname | string | true | none |     | none |
| index | integer | true | none |     | SRT实例id |
| latency | integer | true | none |     | none |
| mode | integer | true | none |     | none |
| password | string | true | none |     | none |
| port | [Port](#schemaport) | true | none |     | none |
| status | string | true | none |     | none |
| streamid | string | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| aesMode | 16  |
| aesMode | 24  |
| aesMode | 32  |
| audio | -1  |
| audio | 6   |
| mode | 1   |
| mode | 2   |
| status | unknown |
| status | ok  |
| status | bad |

<h2 id="tocS_NDI Key">NDI Key</h2>

<a id="schemandi key"></a>
<a id="schema_NDI Key"></a>
<a id="tocSndi key"></a>
<a id="tocsndi key"></a>

```json
{
    "enable": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | enable |
| *anonymous* | enableFixed |
| *anonymous* | streamName |
| *anonymous* | streamNameSuffix |
| *anonymous* | discoveryEnable |
| *anonymous* | discoveryServer |
| *anonymous* | groupName |
| *anonymous* | multicastEnable |
| *anonymous* | multicastIp |
| *anonymous* | multicastNetmask |
| *anonymous* | multicastTtl |
| *anonymous* | ndiAudio |
| *anonymous* | tpm |

<h2 id="tocS_NDI">NDI</h2>

<a id="schemandi"></a>
<a id="schema_NDI"></a>
<a id="tocSndi"></a>
<a id="tocsndi"></a>

```json
{
  "enable": true,
  "enableFixed": true,
  "index": 0,
  "streamName": "string",
  "streamNameSuffix": "string"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| enable | boolean | true | none |     | none |
| enableFixed | boolean | true | none |     | none |
| index | integer | true | none |     | 与get时带入的id一致 |
| streamName | string | true | none |     | none |
| streamNameSuffix | string | true | none |     | none |

<h2 id="tocS_Full NDI Key">Full NDI Key</h2>

<a id="schemafull ndi key"></a>
<a id="schema_Full NDI Key"></a>
<a id="tocSfull ndi key"></a>
<a id="tocsfull ndi key"></a>

```json
"enable"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | channelName |
| *anonymous* | channelNameSuffix |
| *anonymous* | deviceName |
| *anonymous* | dhcp |
| *anonymous* | dns |
| *anonymous* | dynamic |
| *anonymous* | enable |
| *anonymous* | encodeQuality |
| *anonymous* | fallbackIp |
| *anonymous* | fallbackNetmask |
| *anonymous* | gateway |
| *anonymous* | groupName |
| *anonymous* | ip  |
| *anonymous* | multicastEnable |
| *anonymous* | multicastIp |
| *anonymous* | multicastNetmask |
| *anonymous* | netmask |
| *anonymous* | staticIp |
| *anonymous* | ttl |

<h2 id="tocS_Full NDI">Full NDI</h2>

<a id="schemafull ndi"></a>
<a id="schema_Full NDI"></a>
<a id="tocSfull ndi"></a>
<a id="tocsfull ndi"></a>

```json
{
  "channelName": "string",
  "channelNameSuffix": "string",
  "deviceName": "string",
  "dhcp": true,
  "dns": "string",
  "dynamic": true,
  "enable": true,
  "encodeQuality": 0,
  "fallbackIp": "string",
  "fallbackNetmask": "string",
  "gateway": "string",
  "groupName": "string",
  "index": 0,
  "ip": "string",
  "multicastEnable": true,
  "multicastIp": "string",
  "multicastNetmask": "string",
  "netmask": "string",
  "staticIp": "string",
  "ttl": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| channelName | string | true | none |     | NDI Stream Name (HB) |
| channelNameSuffix | string | true | none |     | none |
| deviceName | string | true | none |     | none |
| dhcp | boolean | true | none |     | NDI HB DHCP |
| dns | string | true | none |     | none |
| dynamic | boolean | true | none |     | none |
| enable | boolean | true | none |     | Enable |
| encodeQuality | integer | true | none |     | none |
| fallbackIp | string | true | none |     | NDI HB Fallback Address |
| fallbackNetmask | string | true | none |     | NDI HB Fallback Subnet Mask |
| gateway | string | true | none |     | none |
| groupName | string | true | none |     | none |
| index | integer | true | none |     | none |
| ip  | string | true | none |     | NDI HB Address |
| multicastEnable | boolean | true | none |     | none |
| multicastIp | string | true | none |     | none |
| multicastNetmask | string | true | none |     | none |
| netmask | string | true | none |     | none |
| staticIp | string | true | none |     | none |
| ttl | integer | true | none |     | none |

<h2 id="tocS_Dante Key">Dante Key</h2>

<a id="schemadante key"></a>
<a id="schema_Dante Key"></a>
<a id="tocSdante key"></a>
<a id="tocsdante key"></a>

```json
"enable"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | enable |

<h2 id="tocS_ONVIF Key">ONVIF Key</h2>

<a id="schemaonvif key"></a>
<a id="schema_ONVIF Key"></a>
<a id="tocSonvif key"></a>
<a id="tocsonvif key"></a>

```json
"enable"
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | string | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | enable |
| *anonymous* | port |

<h2 id="tocS_ONVIF">ONVIF</h2>

<a id="schemaonvif"></a>
<a id="schema_ONVIF"></a>
<a id="tocSonvif"></a>
<a id="tocsonvif"></a>

```json
{
  "enable": true,
  "enableFixed": true,
  "index": 0,
  "port": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| enable | boolean | true | none |     | none |
| enableFixed | boolean | true | none |     | none |
| index | integer | true | none |     | none |
| port | integer | true | none |     | none |

<h2 id="tocS_ScreenSaver">ScreenSaver</h2>

<a id="schemascreensaver"></a>
<a id="schema_ScreenSaver"></a>
<a id="tocSscreensaver"></a>
<a id="tocsscreensaver"></a>

```json
{
  "customColor": 16777216,
  "enable": 0,
  "imgName": "string",
  "screensaverMode": 1
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| customColor | integer | true | none |     | #000000-#FFFFFF |
| enable | [Enable](#schemaenable) | true | none |     | none |
| imgName | string | true | none |     | none |
| screensaverMode | integer | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| screensaverMode | 1   |
| screensaverMode | 3   |
| screensaverMode | 4   |

<h2 id="tocS_Color Matrix Color">Color Matrix Color</h2>

<a id="schemacolor matrix color"></a>
<a id="schema_Color Matrix Color"></a>
<a id="tocScolor matrix color"></a>
<a id="tocscolor matrix color"></a>

```json
-1
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| *anonymous* | integer | false | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| *anonymous* | -1  |
| *anonymous* | 0   |
| *anonymous* | 1   |
| *anonymous* | 2   |
| *anonymous* | 3   |
| *anonymous* | 4   |
| *anonymous* | 5   |
| *anonymous* | 6   |

<h2 id="tocS_Color Matrix Hue">Color Matrix Hue</h2>

<a id="schemacolor matrix hue"></a>
<a id="schema_Color Matrix Hue"></a>
<a id="tocScolor matrix hue"></a>
<a id="tocscolor matrix hue"></a>

```json
{
  "blue": -60,
  "cyan": -60,
  "green": -60,
  "id": 0,
  "magenta": -60,
  "red": -60,
  "type": -1,
  "yellow": -60
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| blue | integer | true | none |     | none |
| cyan | integer | true | none |     | none |
| green | integer | true | none |     | none |
| id  | [Lens Channel](#schemalens channel) | true | none |     | none |
| magenta | integer | true | none |     | none |
| red | integer | true | none |     | none |
| type | [Color Matrix Color](#schemacolor matrix color) | true | none |     | 一次只能设置一个颜色的参数，set哪个参数，就填哪个颜色对应的枚举值， |
| yellow | integer | true | none |     | none |

<h2 id="tocS_Color Matrix Satuation">Color Matrix Satuation</h2>

<a id="schemacolor matrix satuation"></a>
<a id="schema_Color Matrix Satuation"></a>
<a id="tocScolor matrix satuation"></a>
<a id="tocscolor matrix satuation"></a>

```json
{
  "blue": 200,
  "cyan": 200,
  "green": 200,
  "id": 0,
  "magenta": 200,
  "red": 200,
  "type": -1,
  "yellow": 200
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| blue | integer | true | none |     | none |
| cyan | integer | true | none |     | none |
| green | integer | true | none |     | none |
| id  | [Lens Channel](#schemalens channel) | true | none |     | none |
| magenta | integer | true | none |     | none |
| red | integer | true | none |     | none |
| type | [Color Matrix Color](#schemacolor matrix color) | true | none |     | 一次只能设置一个颜色的参数，set哪个参数，就填哪个颜色对应的枚举值， |
| yellow | integer | true | none |     | none |

<h2 id="tocS_Business Panel">Business Panel</h2>

<a id="schemabusiness panel"></a>
<a id="schema_Business Panel"></a>
<a id="tocSbusiness panel"></a>
<a id="tocsbusiness panel"></a>

```json
{
  "EventType": 0,
  "bitmap": 0,
  "enable": 0,
  "value": 0,
  "visible": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| EventType | integer | true | none |     | none |
| bitmap | integer | true | none |     | none |
| enable | integer | true | none |     | none |
| value | integer | true | none |     | 参数值 |
| visible | integer | true | none |     | none |

<h2 id="tocS_HDMI">HDMI</h2>

<a id="schemahdmi"></a>
<a id="schema_HDMI"></a>
<a id="tocShdmi"></a>
<a id="tocshdmi"></a>

```json
{
  "hdmiCs": 0,
  "hdmiFormat": 0,
  "hdmiFps": 0,
  "hdmiOutput": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| hdmiCs | integer | true | none |     | Color Space |
| hdmiFormat | integer | true | none |     | Resolution |
| hdmiFps | integer | true | none |     | Frame Rate |
| hdmiOutput | integer | true | none |     | Output Channel |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| hdmiCs | 0   |
| hdmiCs | 1   |
| hdmiCs | 2   |
| hdmiFormat | 0   |
| hdmiFormat | 1   |
| hdmiFormat | 2   |
| hdmiFormat | 3   |
| hdmiFps | 0   |
| hdmiFps | 1   |
| hdmiFps | 2   |
| hdmiFps | 3   |
| hdmiFps | 4   |
| hdmiFps | 5   |
| hdmiFps | 7   |
| hdmiOutput | 0   |
| hdmiOutput | 1   |

<h2 id="tocS_PIP">PIP</h2>

<a id="schemapip"></a>
<a id="schema_PIP"></a>
<a id="tocSpip"></a>
<a id="tocspip"></a>

```json
{
  "Pip": 0,
  "PipPosition": 0,
  "PipSize": 0,
  "PipSource": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| Pip | [Enable](#schemaenable) | true | none |     | none |
| PipPosition | integer | true | none |     | none |
| PipSize | integer | true | none |     | none |
| PipSource | integer | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| PipPosition | 0   |
| PipPosition | 1   |
| PipPosition | 2   |
| PipPosition | 3   |
| PipSize | 0   |
| PipSize | 1   |
| PipSize | 2   |
| PipSource | 0   |
| PipSource | 1   |

<h2 id="tocS_3G SDI">3G SDI</h2>

<a id="schema3g sdi"></a>
<a id="schema_3G SDI"></a>
<a id="tocS3g sdi"></a>
<a id="tocs3g sdi"></a>

```json
{
  "enable": 0,
  "weight": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| enable | [Enable](#schemaenable) | true | none |     | 如果weight=180即开启3路4K视频时，不能开启3G SDI；如果已经开启了3G SDI，在开启第三路4K时，需要关闭3G SDI。 |
| weight | integer | true | none |     | 每一路分辨率视频所对应的权重：4K->60, 1080P/I->30, 720P->15, 360P->5 |

<h2 id="tocS_Language">Language</h2>

<a id="schemalanguage"></a>
<a id="schema_Language"></a>
<a id="tocSlanguage"></a>
<a id="tocslanguage"></a>

```json
{
  "language": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| language | integer | true | none |     | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| language | 0   |
| language | 1   |

<h2 id="tocS_DateTime">DateTime</h2>

<a id="schemadatetime"></a>
<a id="schema_DateTime"></a>
<a id="tocSdatetime"></a>
<a id="tocsdatetime"></a>

```json
{
  "datetimeFormat": 0,
  "deviceClock": "2019-08-24T14:15:22Z",
  "manualDeviceTime": "2019-08-24T14:15:22Z",
  "timeSetting": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| datetimeFormat | integer | true | none | Time Format | none |
| deviceClock | string(date-time) | true | none | Device Time | none |
| manualDeviceTime | string(date-time) | true | none | Manual Set Time | none |
| timeSetting | integer | true | none | Time Sync Mode | none |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| datetimeFormat | 0   |
| datetimeFormat | 1   |
| datetimeFormat | 2   |
| timeSetting | 0   |
| timeSetting | 1   |
| timeSetting | 2   |

<h2 id="tocS_Ethernet Info">Ethernet Info</h2>

<a id="schemaethernet info"></a>
<a id="schema_Ethernet Info"></a>
<a id="tocSethernet info"></a>
<a id="tocsethernet info"></a>

```json
{
  "dns1": "192.168.0.1",
  "dns2": "192.168.0.1",
  "gateway": "192.168.0.1",
  "id": 0,
  "interface": "string",
  "ipaddr": "192.168.0.1",
  "isAutoDns": 0,
  "isDhcp": 0,
  "isEnabled": 0,
  "netmask": "192.168.0.1"
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| dns1 | string(ipv4) | true | none | DNS1 | none |
| dns2 | string(ipv4) | true | none | DNS2 | none |
| gateway | string(ipv4) | true | none | Gateway | none |
| id  | integer | true | none | Network Card ID | none |
| interface | string | true | none | Interface Name | none |
| ipaddr | string(ipv4) | true | none | IP Address | none |
| isAutoDns | [Enable](#schemaenable) | true | none | Auto DNS | none |
| isDhcp | [Enable](#schemaenable) | true | none | DHCP Switch | none |
| isEnabled | [Enable](#schemaenable) | true | none | Network Card Enable | none |
| netmask | string(ipv4) | true | none | Netmask | none |

<h2 id="tocS_TrackArea1">TrackArea1</h2>

<a id="schematrackarea1"></a>
<a id="schema_TrackArea1"></a>
<a id="tocStrackarea1"></a>
<a id="tocstrackarea1"></a>

```json
{
  "LeftDownX": 0,
  "LeftDownY": 2160,
  "LeftTopX": 3840,
  "LeftTopY": 2160,
  "RightDownX": 3840,
  "RightDownY": 2160,
  "RightTopX": 3840,
  "RightTopY": 2160,
  "enable": true,
  "presetId": 0,
  "workMode": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| LeftDownX | integer | false | none | Left Bottom X | X coordinate of left bottom corner of track area, range 0-3840 |
| LeftDownY | integer | false | none | Left Bottom Y | Y coordinate of left bottom corner of track area, range 0-2160 |
| LeftTopX | integer | false | none | Left Top X | X coordinate of left top corner of track area, range 0-3840 |
| LeftTopY | integer | false | none | Left Top Y | Y coordinate of left top corner of track area, range 0-2160 |
| RightDownX | integer | false | none | Right Bottom X | X coordinate of right bottom corner of track area, range 0-3840 |
| RightDownY | integer | false | none | Right Bottom Y | Y coordinate of right bottom corner of track area, range 0-2160 |
| RightTopX | integer | false | none | Right Top X | X coordinate of right top corner of track area, range 0-3840 |
| RightTopY | integer | false | none | Right Top Y | Y coordinate of right top corner of track area, range 0-2160 |
| enable | boolean | false | none | Enable Status | Whether this track area is enabled |
| presetId | integer | false | none | Preset ID | Preset bound to this track area |
| workMode | integer | false | none | Work Mode | Currently selected work mode |

#### 枚举值

| 属性  | 值   |
| --- | --- |
| workMode | 0   |
| workMode | 1   |
| workMode | 2   |

<h2 id="tocS_PanelSetValue">PanelSetValue</h2>

<a id="schemapanelsetvalue"></a>
<a id="schema_PanelSetValue"></a>
<a id="tocSpanelsetvalue"></a>
<a id="tocspanelsetvalue"></a>

```json
{
  "key": "string",
  "value": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| key | string | true | none |     | Function Key |
| value | integer | true | none |     | Value |

<h2 id="tocS_PanelAbility">PanelAbility</h2>

<a id="schemapanelability"></a>
<a id="schema_PanelAbility"></a>
<a id="tocSpanelability"></a>
<a id="tocspanelability"></a>

```json
{
  "component": "string",
  "default": 0,
  "key": "string",
  "order": 0,
  "show": 0,
  "span": 0,
  "feature": {
    "input": 0,
    "max": 0,
    "min": 0,
    "step": 0
  }
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| component | string | true | none |     | 组件类型 |
| default | integer | true | none |     | 默认值 |
| key | string | true | none |     | 功能key |
| order | integer | true | none |     | 排序值 |
| show | integer | true | none |     | 是否显示 |
| span | integer | true | none |     | 栅格大小 |
| feature | object | false | none |     | none |
| » input | integer | true | none |     | none |
| » max | integer | true | none |     | 最大值 |
| » min | integer | true | none |     | 最小值 |
| » step | integer | true | none |     | 步长  |

<h2 id="tocS_PanelGetValue">PanelGetValue</h2>

<a id="schemapanelgetvalue"></a>
<a id="schema_PanelGetValue"></a>
<a id="tocSpanelgetvalue"></a>
<a id="tocspanelgetvalue"></a>

```json
{
  "enabled": 0,
  "key": "string",
  "value": 0
}
```

### 属性

| 名称  | 类型  | 必选  | 约束  | 中文名 | 说明  |
| --- | --- | --- | --- | --- | --- |
| enabled | [Enable](#schemaenable) | true | none |     | 是否启用 |
| key | string | true | none |     | 功能key |
| value | integer | true | none |     | 值   |