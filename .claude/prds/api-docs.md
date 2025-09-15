---
name: api-docs
description: ShareStack 知识付费平台后端API接口规范与设计文档
status: backlog
created: 2025-09-15T01:07:11Z
---

# PRD: ShareStack 后端API接口规范

## 执行摘要

ShareStack知识付费平台后端API接口规范文档，基于产品需求文档(PRD)和原型图分析，设计了完整的RESTful API架构。本文档涵盖用户管理、内容管理、订阅支付、社交互动、数据分析等7大核心模块，共计85+个API接口，确保平台功能的完整实现和高性能运行。

## 问题陈述

### 核心问题
1. **功能覆盖完整性**：需要API接口覆盖PRD中定义的所有核心功能
2. **性能与扩展性**：支持高并发访问和未来功能扩展
3. **数据一致性**：确保多端数据同步和事务安全
4. **安全合规性**：符合数据保护法规和支付安全标准

### 业务价值
- 为前端应用(PC/移动端)提供稳定的数据服务
- 支持知识付费核心业务流程
- 确保平台可扩展性和安全性
- 降低开发和维护成本

## 用户故事

### 创作者用户故事
- 作为创作者，我需要API支持内容创作、发布和管理，以便高效管理我的知识内容
- 作为创作者，我需要订阅管理API，以便设置和管理不同的付费订阅计划
- 作为创作者，我需要数据分析API，以便了解内容表现和收入情况

### 订阅者用户故事
- 作为订阅者，我需要内容发现API，以便找到感兴趣的内容和创作者
- 作为订阅者，我需要支付API，以便安全便捷地订阅付费内容
- 作为订阅者，我需要社交互动API，以便与创作者和其他用户互动

### 平台管理员用户故事
- 作为管理员，我需要管理API，以便进行用户管理、内容审核和平台运营
- 作为管理员，我需要监控API，以便实时了解平台运行状况和用户行为

## 功能需求

### 1. 用户管理系统API

#### 1.1 认证授权接口
```
POST   /api/v1/auth/register              # 用户注册
POST   /api/v1/auth/login                 # 用户登录
POST   /api/v1/auth/logout                # 用户登出
POST   /api/v1/auth/refresh               # Token刷新
POST   /api/v1/auth/forgot-password       # 忘记密码
POST   /api/v1/auth/reset-password        # 重置密码
POST   /api/v1/auth/verify-email          # 邮箱验证
```

**注册接口详细设计**:
```json
POST /api/v1/auth/register
{
  "username": "string(3-30)",
  "email": "string(email)",
  "password": "string(8-128)",
  "user_type": "enum(reader,creator)",
  "agree_terms": "boolean"
}
```

#### 1.2 第三方登录接口
```
POST   /api/v1/auth/social/wechat         # 微信登录
POST   /api/v1/auth/social/google         # Google登录
POST   /api/v1/auth/social/github         # GitHub登录
GET    /api/v1/auth/social/{provider}/callback  # OAuth回调
```

#### 1.3 用户资料管理
```
GET    /api/v1/users/profile              # 获取个人资料
PUT    /api/v1/users/profile              # 更新个人资料
POST   /api/v1/users/avatar               # 上传头像
GET    /api/v1/users/{id}                 # 获取用户公开信息
DELETE /api/v1/users/account              # 注销账号
```

### 2. 内容管理系统API

#### 2.1 内容创作与编辑
```
POST   /api/v1/articles                   # 创建文章
GET    /api/v1/articles                   # 获取文章列表
GET    /api/v1/articles/{id}              # 获取文章详情
PUT    /api/v1/articles/{id}              # 更新文章
DELETE /api/v1/articles/{id}              # 删除文章
POST   /api/v1/articles/{id}/publish      # 发布文章
POST   /api/v1/articles/{id}/unpublish    # 取消发布
```

**文章创建接口详细设计**:
```json
POST /api/v1/articles
{
  "title": "string(1-200)",
  "content": "string",
  "excerpt": "string(0-500)",
  "cover_image": "string(url)",
  "tags": ["string"],
  "category_id": "integer",
  "visibility": "enum(public,subscribers,paid)",
  "allow_comments": "boolean",
  "scheduled_at": "datetime",
  "seo_title": "string(0-60)",
  "seo_description": "string(0-160)",
  "custom_url": "string(slug)"
}
```

#### 2.2 草稿和版本管理
```
GET    /api/v1/articles/drafts            # 获取草稿列表
POST   /api/v1/articles/{id}/autosave     # 自动保存
GET    /api/v1/articles/{id}/versions     # 获取版本历史
POST   /api/v1/articles/{id}/restore      # 恢复版本
```

#### 2.3 媒体文件管理
```
POST   /api/v1/uploads/image              # 图片上传
POST   /api/v1/uploads/video              # 视频上传
POST   /api/v1/uploads/audio              # 音频上传
GET    /api/v1/uploads/{id}               # 获取文件信息
DELETE /api/v1/uploads/{id}               # 删除文件
```

#### 2.4 分类标签管理
```
GET    /api/v1/categories                 # 获取分类列表
POST   /api/v1/categories                 # 创建分类
PUT    /api/v1/categories/{id}            # 更新分类
DELETE /api/v1/categories/{id}            # 删除分类
GET    /api/v1/tags                       # 获取标签列表
POST   /api/v1/tags                       # 创建标签
```

### 3. 订阅与支付系统API

#### 3.1 订阅计划管理
```
GET    /api/v1/subscription-plans         # 获取订阅计划
POST   /api/v1/subscription-plans         # 创建订阅计划
PUT    /api/v1/subscription-plans/{id}    # 更新订阅计划
DELETE /api/v1/subscription-plans/{id}    # 删除订阅计划
```

**订阅计划接口详细设计**:
```json
POST /api/v1/subscription-plans
{
  "name": "string(1-100)",
  "description": "string(0-500)",
  "price": "decimal(10,2)",
  "currency": "enum(CNY,USD)",
  "billing_cycle": "enum(monthly,yearly)",
  "features": ["string"],
  "trial_days": "integer",
  "is_active": "boolean"
}
```

#### 3.2 订阅管理
```
POST   /api/v1/subscriptions             # 创建订阅
GET    /api/v1/subscriptions             # 获取订阅列表
GET    /api/v1/subscriptions/{id}        # 获取订阅详情
PUT    /api/v1/subscriptions/{id}/cancel # 取消订阅
POST   /api/v1/subscriptions/{id}/renew  # 续费订阅
```

#### 3.3 支付处理
```
POST   /api/v1/payments/create            # 创建支付订单
GET    /api/v1/payments/{id}              # 获取支付状态
POST   /api/v1/payments/{id}/confirm      # 确认支付
POST   /api/v1/payments/webhooks/wechat   # 微信支付回调
POST   /api/v1/payments/webhooks/alipay   # 支付宝回调
POST   /api/v1/payments/webhooks/stripe   # Stripe回调
```

#### 3.4 财务管理
```
GET    /api/v1/earnings                   # 获取收入统计
GET    /api/v1/earnings/transactions      # 获取交易记录
POST   /api/v1/withdrawals                # 申请提现
GET    /api/v1/withdrawals                # 获取提现记录
GET    /api/v1/tax-info                   # 获取税务信息
PUT    /api/v1/tax-info                   # 更新税务信息
```

### 4. 内容发现与推荐API

#### 4.1 搜索功能
```
GET    /api/v1/search                     # 全文搜索
GET    /api/v1/search/suggestions         # 搜索建议
GET    /api/v1/search/trending            # 热门搜索
POST   /api/v1/search/history             # 记录搜索历史
```

**搜索接口详细设计**:
```json
GET /api/v1/search?q=keyword&type=articles&category=tech&sort=relevance&page=1&limit=20
Response:
{
  "success": true,
  "data": {
    "results": [],
    "total": 150,
    "page": 1,
    "limit": 20,
    "suggestions": ["machine learning", "web development"]
  }
}
```

#### 4.2 推荐算法
```
GET    /api/v1/recommendations/feed       # 个性化推荐
GET    /api/v1/recommendations/trending   # 热门内容
GET    /api/v1/recommendations/creators   # 推荐创作者
GET    /api/v1/recommendations/similar    # 相似内容
```

#### 4.3 内容发现
```
GET    /api/v1/feed/home                  # 首页信息流
GET    /api/v1/feed/following             # 关注动态
GET    /api/v1/feed/category/{id}         # 分类内容
GET    /api/v1/rankings/articles          # 文章排行榜
GET    /api/v1/rankings/creators          # 创作者排行榜
```

### 5. 社交与互动API

#### 5.1 社交功能
```
POST   /api/v1/follows                    # 关注用户
DELETE /api/v1/follows/{user_id}          # 取消关注
GET    /api/v1/users/{id}/followers       # 获取粉丝列表
GET    /api/v1/users/{id}/following       # 获取关注列表
```

#### 5.2 互动功能
```
POST   /api/v1/articles/{id}/like         # 点赞文章
DELETE /api/v1/articles/{id}/like         # 取消点赞
POST   /api/v1/articles/{id}/bookmark     # 收藏文章
DELETE /api/v1/articles/{id}/bookmark     # 取消收藏
GET    /api/v1/users/bookmarks            # 获取收藏列表
```

#### 5.3 评论系统
```
POST   /api/v1/articles/{id}/comments     # 创建评论
GET    /api/v1/articles/{id}/comments     # 获取评论列表
PUT    /api/v1/comments/{id}              # 更新评论
DELETE /api/v1/comments/{id}              # 删除评论
POST   /api/v1/comments/{id}/reply        # 回复评论
POST   /api/v1/comments/{id}/like         # 点赞评论
POST   /api/v1/comments/{id}/report       # 举报评论
```

**评论创建接口详细设计**:
```json
POST /api/v1/articles/{id}/comments
{
  "content": "string(1-1000)",
  "parent_id": "integer",
  "mentions": ["integer"]
}
```

### 6. 数据分析与洞察API

#### 6.1 创作者分析
```
GET    /api/v1/analytics/overview         # 数据概览
GET    /api/v1/analytics/articles         # 文章分析
GET    /api/v1/analytics/revenue          # 收入分析
GET    /api/v1/analytics/audience         # 受众分析
GET    /api/v1/analytics/traffic          # 流量分析
```

#### 6.2 平台数据监控
```
GET    /api/v1/admin/analytics/platform   # 平台数据概览
GET    /api/v1/admin/analytics/users      # 用户行为分析
GET    /api/v1/admin/analytics/content    # 内容统计分析
GET    /api/v1/admin/analytics/revenue    # 平台收入分析
```

### 7. 通知系统API

#### 7.1 通知管理
```
GET    /api/v1/notifications              # 获取通知列表
PUT    /api/v1/notifications/{id}/read    # 标记已读
PUT    /api/v1/notifications/read-all     # 全部标记已读
DELETE /api/v1/notifications/{id}         # 删除通知
```

#### 7.2 推送设置
```
GET    /api/v1/notification-settings      # 获取推送设置
PUT    /api/v1/notification-settings      # 更新推送设置
POST   /api/v1/push-tokens                # 注册推送Token
DELETE /api/v1/push-tokens/{token}        # 删除推送Token
```

## 非功能性需求

### 性能要求
- **响应时间**: 95%的API请求响应时间<500ms
- **并发处理**: 支持10,000+并发用户
- **数据吞吐**: 支持每秒1,000+次API调用
- **可用性**: 99.9%服务可用时间

### 安全要求
- **认证机制**: JWT Token + Refresh Token
- **权限控制**: RBAC基于角色的访问控制
- **数据加密**: HTTPS + 数据库敏感字段加密
- **输入验证**: 所有输入参数严格校验
- **API限流**: 基于用户和IP的请求频率限制

### 数据格式规范

#### 统一响应格式
```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 100,
      "pages": 5
    },
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### 错误响应格式
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "参数验证失败",
    "details": {
      "field": "email",
      "message": "邮箱格式不正确"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

## 数据库设计

### 核心数据表
1. **users** - 用户基本信息
2. **user_profiles** - 用户扩展资料
3. **articles** - 文章内容
4. **categories** - 内容分类
5. **tags** - 标签管理
6. **subscription_plans** - 订阅计划
7. **subscriptions** - 订阅记录
8. **payments** - 支付记录
9. **comments** - 评论系统
10. **follows** - 关注关系
11. **likes** - 点赞记录
12. **bookmarks** - 收藏记录
13. **notifications** - 通知系统
14. **analytics_events** - 分析事件

### 索引优化策略
- 用户ID、文章ID等主键自动索引
- 创建时间、更新时间的时间索引
- 邮箱、用户名的唯一索引
- 文章标题、内容的全文索引
- 复合索引优化查询性能

## 成功标准

### 技术指标
- [ ] API响应时间P95 < 500ms
- [ ] 系统可用性 > 99.9%
- [ ] 支持10,000+并发用户
- [ ] 数据一致性100%保证

### 功能指标
- [ ] 85+个API接口完整实现
- [ ] 覆盖PRD中100%核心功能
- [ ] 支持PC端和移动端全部功能
- [ ] 通过安全测试和性能测试

### 业务指标
- [ ] 支持平台MVP版本上线
- [ ] 满足第一批1000+用户需求
- [ ] 支持创作者收入管理
- [ ] 实现完整的付费订阅流程

## 约束条件

### 技术约束
- 使用FastAPI + Python 3.11+开发
- MySQL 8.0作为主数据库
- Redis 7作为缓存和队列
- 遵循RESTful API设计规范

### 时间约束
- MVP版本API开发周期3个月
- 第一阶段覆盖核心功能80%
- 第二阶段完善高级功能和优化

### 资源约束
- 后端开发团队2人
- 测试工程师1人
- DevOps工程师1人

## 范围外内容

明确不包含在此API规范中的功能：
- 前端界面设计和实现
- 移动应用开发
- 第三方服务集成(支付网关配置)
- 运维部署和监控配置
- 数据迁移和初始化脚本

## 依赖关系

### 外部依赖
- 微信支付/支付宝支付接口
- 邮件服务提供商(SendGrid/阿里云邮件)
- 短信服务提供商
- CDN服务(CloudFlare)
- 对象存储服务(AWS S3/阿里云OSS)

### 内部依赖
- 数据库设计完成
- 认证授权系统实现
- 文件上传服务就绪
- 推荐算法服务开发

---

**文档版本**: v1.0
**创建时间**: 2025-09-15
**最后更新**: 2025-09-15
**负责团队**: 后端开发团队

*本API规范文档基于ShareStack知识付费平台产品需求文档(PRD)和原型图分析制定，为后端API开发提供完整的技术规范和实施指南。*