# ShareStack API 文档

## 概述

ShareStack 知识付费平台后端 API 文档集合，提供完整的接口规范、使用示例和最佳实践指南。

## 文档目录

### 核心模块

- [用户管理 API](./user-management.md) - 认证授权、第三方登录、用户资料管理
- [内容管理 API](./content-management.md) - 文章创作、媒体上传、分类标签
- 订阅支付 API - 订阅计划、支付处理、财务管理（开发中）
- 社交互动 API - 评论系统、点赞收藏、关注功能（开发中）
- 数据分析 API - 用户行为、内容统计、收入分析（开发中）

## 快速开始

### 基础信息

- **API 版本**: v1
- **基础 URL**: `https://api.sharestack.com/api/v1`
- **认证方式**: Bearer Token (JWT)
- **响应格式**: JSON

### 认证流程

1. **用户注册**
   ```bash
   curl -X POST "/api/v1/auth/register" \
     -H "Content-Type: application/json" \
     -d '{"username":"user","email":"user@example.com","password":"pass123","user_type":"reader","agree_terms":true}'
   ```

2. **用户登录**
   ```bash
   curl -X POST "/api/v1/auth/login" \
     -H "Content-Type: application/json" \
     -d '{"email":"user@example.com","password":"pass123"}'
   ```

3. **使用Token访问受保护接口**
   ```bash
   curl -X GET "/api/v1/users/profile" \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
   ```

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

## 接口统计

| 模块 | 接口数量 | 完成状态 |
|------|----------|----------|
| 用户管理 | 16个 | ✅ 完成 |
| 内容管理 | 22个 | ✅ 完成 |
| 订阅支付 | 12个 | 🚧 开发中 |
| 社交互动 | 10个 | 🚧 开发中 |
| 数据分析 | 8个 | 🚧 开发中 |
| 通知系统 | 6个 | 🚧 开发中 |
| **总计** | **74个** | **51%** |

## 安全说明

### 认证机制
- JWT Token认证，访问令牌有效期1小时
- 刷新令牌有效期30天，支持自动续期
- 登出时令牌立即失效

### API限流
- 注册接口：每IP每小时最多5次
- 登录接口：每IP每分钟最多10次
- 一般API：每用户每分钟最多1000次

### 数据安全
- 所有API使用HTTPS加密传输
- 敏感数据加密存储
- 支持GDPR数据保护合规

## 开发工具

### Postman集合
- [ShareStack API Postman Collection](./postman/sharestack-api.json)（待创建）

### OpenAPI规范
- [OpenAPI 3.0 Specification](./openapi/sharestack-api.yaml)（待创建）

### SDK客户端
- JavaScript SDK（规划中）
- Python SDK（规划中）
- React Hook库（规划中）

## 更新日志

### v1.0.0 (2025-09-15)
- ✅ 完成用户管理API文档
  - 认证授权接口 (7个)
  - 第三方登录接口 (4个)
  - 用户资料管理接口 (5个)
- ✅ 完成内容管理API文档
  - 文章CRUD接口 (7个)
  - 草稿和版本管理接口 (4个)
  - 媒体文件管理接口 (5个)
  - 分类标签管理接口 (6个)
- 📝 建立API文档结构和规范
- 🔒 定义安全策略和最佳实践

### 即将发布
- 订阅支付API文档 (预计 2025-09-16)
- 社交互动API文档 (预计 2025-09-17)
- 数据分析API文档 (预计 2025-09-18)

## 联系我们

- **API支持**: api-support@sharestack.com
- **技术文档**: docs@sharestack.com
- **GitHub Issues**: [sharestack/api-issues](https://github.com/rcnn/sharestack/issues)

---

**文档版本**: v1.0.0
**维护团队**: ShareStack 后端开发团队
**最后更新**: 2025-09-15