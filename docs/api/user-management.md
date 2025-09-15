# ShareStack 用户管理 API 文档

## 概述

ShareStack 用户管理模块提供完整的用户认证、授权、资料管理和第三方登录功能。本文档遵循 OpenAPI 3.0 规范，为所有用户相关接口提供详细的规范说明。

### 基础信息

- **API 版本**: v1
- **基础 URL**: `https://api.sharestack.com/api/v1`
- **认证方式**: Bearer Token (JWT)
- **响应格式**: JSON

### 通用响应格式

#### 成功响应
```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### 错误响应
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "ERROR_CODE",
    "message": "错误描述",
    "details": {}
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

### 通用错误码

| 错误码 | HTTP状态码 | 描述 |
|--------|-----------|------|
| VALIDATION_ERROR | 400 | 请求参数验证失败 |
| UNAUTHORIZED | 401 | 认证失败或令牌无效 |
| FORBIDDEN | 403 | 权限不足 |
| NOT_FOUND | 404 | 资源不存在 |
| CONFLICT | 409 | 资源冲突 |
| RATE_LIMIT_EXCEEDED | 429 | 请求频率超限 |
| INTERNAL_ERROR | 500 | 服务器内部错误 |

---

## 1. 认证授权接口

### 1.1 用户注册

**接口路径**: `POST /auth/register`

**描述**: 注册新用户账号，支持普通用户和创作者两种类型。

#### 请求参数

```json
{
  "username": "string",      // 用户名，3-30字符，字母数字下划线
  "email": "string",         // 邮箱地址，必须是有效格式
  "password": "string",      // 密码，8-128字符，包含字母数字特殊字符
  "user_type": "string",     // 用户类型：reader|creator
  "agree_terms": "boolean"   // 是否同意服务条款，必须为true
}
```

#### 响应示例

**成功响应 (201)**:
```json
{
  "success": true,
  "data": {
    "user_id": 12345,
    "username": "john_doe",
    "email": "john@example.com",
    "user_type": "creator",
    "email_verified": false,
    "created_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

**错误响应 (400)**:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "注册信息验证失败",
    "details": {
      "email": "邮箱格式不正确",
      "password": "密码强度不够"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X POST "https://api.sharestack.com/api/v1/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "john_doe",
    "email": "john@example.com",
    "password": "SecurePass123!",
    "user_type": "creator",
    "agree_terms": true
  }'
```

#### JavaScript 示例

```javascript
const response = await fetch('/api/v1/auth/register', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    username: 'john_doe',
    email: 'john@example.com',
    password: 'SecurePass123!',
    user_type: 'creator',
    agree_terms: true
  })
});

const result = await response.json();
if (result.success) {
  console.log('注册成功:', result.data);
} else {
  console.error('注册失败:', result.error.message);
}
```

---

### 1.2 用户登录

**接口路径**: `POST /auth/login`

**描述**: 用户登录验证，返回访问令牌和刷新令牌。

#### 请求参数

```json
{
  "email": "string",         // 邮箱地址或用户名
  "password": "string",      // 密码
  "remember_me": "boolean"   // 是否记住登录状态（可选）
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "user_id": 12345,
      "username": "john_doe",
      "email": "john@example.com",
      "user_type": "creator",
      "email_verified": true,
      "avatar": "https://cdn.sharestack.com/avatars/12345.jpg"
    }
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X POST "https://api.sharestack.com/api/v1/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "SecurePass123!",
    "remember_me": true
  }'
```

#### JavaScript 示例

```javascript
const response = await fetch('/api/v1/auth/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    email: 'john@example.com',
    password: 'SecurePass123!',
    remember_me: true
  })
});

const result = await response.json();
if (result.success) {
  // 保存令牌
  localStorage.setItem('access_token', result.data.access_token);
  localStorage.setItem('refresh_token', result.data.refresh_token);
} else {
  console.error('登录失败:', result.error.message);
}
```

---

### 1.3 用户登出

**接口路径**: `POST /auth/logout`

**描述**: 用户登出，使当前令牌失效。

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "登出成功"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X POST "https://api.sharestack.com/api/v1/auth/logout" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### 1.4 令牌刷新

**接口路径**: `POST /auth/refresh`

**描述**: 使用刷新令牌获取新的访问令牌。

#### 请求参数

```json
{
  "refresh_token": "string"  // 刷新令牌
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.5 忘记密码

**接口路径**: `POST /auth/forgot-password`

**描述**: 发送密码重置邮件到用户邮箱。

#### 请求参数

```json
{
  "email": "string"  // 用户邮箱地址
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "密码重置邮件已发送",
    "email": "john@example.com"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.6 重置密码

**接口路径**: `POST /auth/reset-password`

**描述**: 使用重置令牌设置新密码。

#### 请求参数

```json
{
  "reset_token": "string",   // 重置令牌（来自邮件链接）
  "new_password": "string"   // 新密码
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "密码重置成功"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.7 邮箱验证

**接口路径**: `POST /auth/verify-email`

**描述**: 验证用户邮箱地址。

#### 请求参数

```json
{
  "verification_token": "string"  // 验证令牌（来自邮件链接）
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "邮箱验证成功",
    "email_verified": true
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 2. 第三方登录接口

### 2.1 微信登录

**接口路径**: `POST /auth/social/wechat`

**描述**: 通过微信OAuth进行用户登录或注册。

#### 请求参数

```json
{
  "code": "string",          // 微信授权码
  "state": "string"          // 防CSRF状态参数
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "user_id": 12345,
      "username": "wechat_user_12345",
      "email": null,
      "user_type": "reader",
      "avatar": "https://wx.qlogo.cn/mmopen/...",
      "wechat_info": {
        "openid": "oqw4ft1J-VQb1Q5qNr1Q5qNr1Q5q",
        "nickname": "微信用户",
        "unionid": "oqw4ft1J-VQb1Q5qNr1Q5qNr1Q5q"
      }
    },
    "is_new_user": false
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X POST "https://api.sharestack.com/api/v1/auth/social/wechat" \
  -H "Content-Type: application/json" \
  -d '{
    "code": "061QA7ll2yUOYm0eEvml2Z4kjp3QA7lh",
    "state": "random_state_string"
  }'
```

---

### 2.2 Google登录

**接口路径**: `POST /auth/social/google`

**描述**: 通过Google OAuth进行用户登录或注册。

#### 请求参数

```json
{
  "id_token": "string"       // Google ID Token
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "user_id": 12346,
      "username": "google_user_12346",
      "email": "user@gmail.com",
      "user_type": "reader",
      "email_verified": true,
      "avatar": "https://lh3.googleusercontent.com/...",
      "google_info": {
        "google_id": "104958547875123456789",
        "email": "user@gmail.com",
        "name": "John Doe"
      }
    },
    "is_new_user": true
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 2.3 GitHub登录

**接口路径**: `POST /auth/social/github`

**描述**: 通过GitHub OAuth进行用户登录或注册。

#### 请求参数

```json
{
  "code": "string",          // GitHub授权码
  "state": "string"          // 防CSRF状态参数
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "user_id": 12347,
      "username": "github_user_12347",
      "email": "developer@github.com",
      "user_type": "creator",
      "email_verified": true,
      "avatar": "https://avatars.githubusercontent.com/...",
      "github_info": {
        "github_id": 12345678,
        "login": "developer",
        "name": "GitHub Developer"
      }
    },
    "is_new_user": false
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 2.4 OAuth回调处理

**接口路径**: `GET /auth/social/{provider}/callback`

**描述**: 处理第三方OAuth回调，支持微信、Google、GitHub等提供商。

#### 路径参数

- `provider`: 第三方提供商 (`wechat` | `google` | `github`)

#### 查询参数

```
code: 授权码
state: 状态参数
```

#### 响应示例

**成功响应 (302)**:
重定向到前端页面，携带临时令牌：
```
Location: https://sharestack.com/auth/callback?token=temp_token_12345
```

---

## 3. 用户资料管理接口

### 3.1 获取个人资料

**接口路径**: `GET /users/profile`

**描述**: 获取当前用户的详细资料信息。

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "user_id": 12345,
    "username": "john_doe",
    "email": "john@example.com",
    "user_type": "creator",
    "display_name": "John Doe",
    "bio": "分享技术和生活感悟的创作者",
    "avatar": "https://cdn.sharestack.com/avatars/12345.jpg",
    "cover_image": "https://cdn.sharestack.com/covers/12345.jpg",
    "location": "北京, 中国",
    "website": "https://johndoe.com",
    "social_links": {
      "twitter": "johndoe",
      "github": "johndoe",
      "linkedin": "johndoe"
    },
    "email_verified": true,
    "created_at": "2025-01-15T10:30:00Z",
    "updated_at": "2025-09-15T01:07:11Z",
    "stats": {
      "followers_count": 1250,
      "following_count": 180,
      "articles_count": 45,
      "likes_received": 2890
    },
    "subscription_info": {
      "is_creator": true,
      "monthly_price": 29.99,
      "yearly_price": 299.99,
      "subscribers_count": 89
    }
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X GET "https://api.sharestack.com/api/v1/users/profile" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### 3.2 更新个人资料

**接口路径**: `PUT /users/profile`

**描述**: 更新当前用户的资料信息。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "display_name": "string",     // 显示名称，可选
  "bio": "string",              // 个人简介，可选，最长500字符
  "location": "string",         // 所在地，可选
  "website": "string",          // 个人网站，可选，需要有效URL格式
  "social_links": {             // 社交链接，可选
    "twitter": "string",
    "github": "string",
    "linkedin": "string"
  },
  "subscription_settings": {    // 订阅设置，仅创作者可用
    "monthly_price": "number",  // 月费价格
    "yearly_price": "number",   // 年费价格
    "description": "string"     // 订阅说明
  }
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "资料更新成功",
    "updated_fields": ["display_name", "bio", "website"]
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### JavaScript 示例

```javascript
const updateProfile = async (profileData) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch('/api/v1/users/profile', {
    method: 'PUT',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(profileData)
  });

  const result = await response.json();
  return result;
};

// 使用示例
const result = await updateProfile({
  display_name: 'John Doe',
  bio: '分享技术和生活感悟的创作者',
  website: 'https://johndoe.com',
  social_links: {
    twitter: 'johndoe',
    github: 'johndoe'
  }
});
```

---

### 3.3 上传头像

**接口路径**: `POST /users/avatar`

**描述**: 上传用户头像图片。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

#### 请求参数

```
avatar: File  // 头像文件，支持JPG/PNG格式，最大5MB
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "头像上传成功",
    "avatar_url": "https://cdn.sharestack.com/avatars/12345.jpg",
    "thumbnail_url": "https://cdn.sharestack.com/avatars/thumbs/12345.jpg"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X POST "https://api.sharestack.com/api/v1/users/avatar" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -F "avatar=@/path/to/avatar.jpg"
```

#### JavaScript 示例

```javascript
const uploadAvatar = async (file) => {
  const token = localStorage.getItem('access_token');
  const formData = new FormData();
  formData.append('avatar', file);

  const response = await fetch('/api/v1/users/avatar', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`
    },
    body: formData
  });

  const result = await response.json();
  return result;
};

// 使用示例
const fileInput = document.getElementById('avatar-input');
fileInput.addEventListener('change', async (event) => {
  const file = event.target.files[0];
  if (file) {
    const result = await uploadAvatar(file);
    if (result.success) {
      console.log('头像上传成功:', result.data.avatar_url);
    }
  }
});
```

---

### 3.4 获取用户公开信息

**接口路径**: `GET /users/{id}`

**描述**: 获取指定用户的公开资料信息。

#### 路径参数

- `id`: 用户ID

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "user_id": 12345,
    "username": "john_doe",
    "display_name": "John Doe",
    "bio": "分享技术和生活感悟的创作者",
    "avatar": "https://cdn.sharestack.com/avatars/12345.jpg",
    "cover_image": "https://cdn.sharestack.com/covers/12345.jpg",
    "location": "北京, 中国",
    "website": "https://johndoe.com",
    "social_links": {
      "twitter": "johndoe",
      "github": "johndoe"
    },
    "user_type": "creator",
    "created_at": "2025-01-15T10:30:00Z",
    "stats": {
      "followers_count": 1250,
      "articles_count": 45,
      "likes_received": 2890
    },
    "subscription_info": {
      "is_creator": true,
      "monthly_price": 29.99,
      "yearly_price": 299.99,
      "description": "获得所有付费内容的访问权限"
    },
    "is_following": false,
    "is_subscribed": false
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 3.5 注销账号

**接口路径**: `DELETE /users/account`

**描述**: 永久删除用户账号及相关数据。

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 请求参数

```json
{
  "password": "string",      // 当前密码确认
  "reason": "string"         // 注销原因，可选
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "账号注销成功",
    "scheduled_deletion_date": "2025-10-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 安全考虑

### 密码安全策略

1. **密码复杂度要求**:
   - 最少8个字符，最多128个字符
   - 必须包含大小写字母、数字和特殊字符
   - 不能是常见弱密码

2. **密码存储**:
   - 使用bcrypt算法进行哈希加密
   - 加盐强度至少12轮

3. **登录安全**:
   - 连续5次登录失败后锁定账号15分钟
   - 记录登录日志和异常登录检测

### JWT令牌安全

1. **令牌设计**:
   - 访问令牌有效期1小时
   - 刷新令牌有效期30天
   - 使用HS256算法签名

2. **令牌撤销**:
   - 支持令牌黑名单机制
   - 登出时立即失效令牌
   - 密码变更时撤销所有令牌

### API限流策略

1. **注册接口**: 每IP每小时最多5次
2. **登录接口**: 每IP每分钟最多10次
3. **密码重置**: 每邮箱每小时最多3次
4. **一般API**: 每用户每分钟最多1000次

### 数据保护

1. **敏感信息**:
   - 邮箱地址部分隐藏显示
   - 手机号码部分隐藏显示
   - 第三方平台ID不对外暴露

2. **GDPR合规**:
   - 支持数据导出功能
   - 支持数据删除权利
   - 明确数据使用授权

---

## 最佳实践

### 前端集成建议

1. **令牌管理**:
   ```javascript
   // 自动刷新令牌示例
   const refreshToken = async () => {
     const refreshToken = localStorage.getItem('refresh_token');
     const response = await fetch('/api/v1/auth/refresh', {
       method: 'POST',
       headers: { 'Content-Type': 'application/json' },
       body: JSON.stringify({ refresh_token: refreshToken })
     });

     if (response.ok) {
       const result = await response.json();
       localStorage.setItem('access_token', result.data.access_token);
       localStorage.setItem('refresh_token', result.data.refresh_token);
       return result.data.access_token;
     } else {
       // 刷新失败，跳转登录页
       window.location.href = '/login';
     }
   };
   ```

2. **错误处理**:
   ```javascript
   const apiCall = async (url, options = {}) => {
     const token = localStorage.getItem('access_token');

     const response = await fetch(url, {
       ...options,
       headers: {
         'Authorization': `Bearer ${token}`,
         'Content-Type': 'application/json',
         ...options.headers
       }
     });

     if (response.status === 401) {
       // 尝试刷新令牌
       const newToken = await refreshToken();
       if (newToken) {
         // 重试原请求
         return fetch(url, {
           ...options,
           headers: {
             'Authorization': `Bearer ${newToken}`,
             'Content-Type': 'application/json',
             ...options.headers
           }
         });
       }
     }

     return response;
   };
   ```

### 第三方登录集成

1. **微信登录流程**:
   ```javascript
   // 前端跳转到微信授权页面
   const wechatLogin = () => {
     const appId = 'your_wechat_app_id';
     const redirectUri = encodeURIComponent('https://sharestack.com/auth/callback');
     const state = generateRandomState();

     window.location.href = `https://open.weixin.qq.com/connect/oauth2/authorize?appid=${appId}&redirect_uri=${redirectUri}&response_type=code&scope=snsapi_userinfo&state=${state}`;
   };
   ```

2. **Google登录流程**:
   ```javascript
   // 使用Google Sign-In JavaScript库
   const initGoogleSignIn = () => {
     gapi.load('auth2', () => {
       gapi.auth2.init({
         client_id: 'your_google_client_id'
       });
     });
   };

   const googleSignIn = () => {
     const authInstance = gapi.auth2.getAuthInstance();
     authInstance.signIn().then(googleUser => {
       const idToken = googleUser.getAuthResponse().id_token;

       // 发送到后端验证
       fetch('/api/v1/auth/social/google', {
         method: 'POST',
         headers: { 'Content-Type': 'application/json' },
         body: JSON.stringify({ id_token: idToken })
       });
     });
   };
   ```

---

## OpenAPI 3.0 规范

```yaml
openapi: 3.0.3
info:
  title: ShareStack 用户管理 API
  description: ShareStack 知识付费平台用户管理模块 API 接口
  version: 1.0.0
  contact:
    name: API Support
    email: api-support@sharestack.com
servers:
  - url: https://api.sharestack.com/api/v1
    description: Production server
  - url: https://staging-api.sharestack.com/api/v1
    description: Staging server

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    User:
      type: object
      properties:
        user_id:
          type: integer
          example: 12345
        username:
          type: string
          example: "john_doe"
        email:
          type: string
          format: email
          example: "john@example.com"
        user_type:
          type: string
          enum: [reader, creator]
        display_name:
          type: string
          example: "John Doe"
        bio:
          type: string
          example: "分享技术和生活感悟的创作者"
        avatar:
          type: string
          format: uri
          example: "https://cdn.sharestack.com/avatars/12345.jpg"

    Error:
      type: object
      properties:
        code:
          type: string
          example: "VALIDATION_ERROR"
        message:
          type: string
          example: "参数验证失败"
        details:
          type: object

security:
  - bearerAuth: []

paths:
  /auth/register:
    post:
      summary: 用户注册
      tags: [认证授权]
      security: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [username, email, password, user_type, agree_terms]
              properties:
                username:
                  type: string
                  minLength: 3
                  maxLength: 30
                email:
                  type: string
                  format: email
                password:
                  type: string
                  minLength: 8
                  maxLength: 128
                user_type:
                  type: string
                  enum: [reader, creator]
                agree_terms:
                  type: boolean
      responses:
        '201':
          description: 注册成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/User'
        '400':
          description: 参数验证失败
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: false
                  error:
                    $ref: '#/components/schemas/Error'
```

---

## 变更日志

### v1.0.0 (2025-09-15)
- 初始版本发布
- 完成认证授权接口设计 (7个接口)
- 完成第三方登录接口设计 (4个接口)
- 完成用户资料管理接口设计 (5个接口)
- 添加完整的安全策略和最佳实践说明
- 提供OpenAPI 3.0规范定义

---

**文档版本**: v1.0.0
**创建时间**: 2025-09-15
**最后更新**: 2025-09-15
**负责团队**: 后端开发团队

*本API文档基于ShareStack知识付费平台PRD需求制定，为用户管理模块提供完整的接口规范和实施指南。*