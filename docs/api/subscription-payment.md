# ShareStack 订阅支付 API 文档

## 概述

ShareStack 订阅支付模块提供完整的订阅计划管理、支付处理、财务统计等核心付费功能。本文档涵盖多支付渠道集成（微信支付、支付宝、Stripe）、安全机制设计和财务合规要求，遵循 OpenAPI 3.0 规范。

### 基础信息

- **API 版本**: v1
- **基础 URL**: `https://api.sharestack.com/api/v1`
- **认证方式**: Bearer Token (JWT)
- **响应格式**: JSON
- **支付渠道**: 微信支付、支付宝、Stripe

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

### 支付安全机制

#### 数据加密传输
- 所有支付相关接口强制使用 HTTPS TLS 1.3
- 敏感数据字段使用 AES-256 加密存储
- 支付密钥独立管理，定期轮换

#### 签名验证机制
```bash
# 签名计算方式
signature = HMAC-SHA256(secret_key, timestamp + method + uri + body)

# 请求头示例
X-Signature: sha256=a1b2c3d4e5f6...
X-Timestamp: 1726361431
X-Nonce: random_string
```

#### 防重复支付设计
- 客户端生成唯一 `idempotency_key`
- 服务端 15 分钟内相同 key 返回相同结果
- 支付状态机严格控制状态转换

#### 风控系统集成
- 实时风险评分（用户行为、设备指纹、交易模式）
- 可疑交易自动拦截（异地登录、大额支付、频繁退款）
- 反洗钱（AML）合规检查

---

## 1. 订阅计划管理接口

### 1.1 获取订阅计划列表

**接口描述**: 获取创作者的所有订阅计划，支持分页和筛选

```http
GET /api/v1/subscription-plans
```

**请求参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| creator_id | integer | 否 | 创作者ID（管理员可指定） |
| status | string | 否 | 状态筛选: `active`, `inactive`, `draft` |
| page | integer | 否 | 页码，默认1 |
| limit | integer | 否 | 每页数量，默认20，最大100 |

**请求示例**:
```bash
curl -X GET "/api/v1/subscription-plans?status=active&page=1&limit=10" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "plans": [
      {
        "id": 1,
        "name": "基础订阅",
        "description": "访问所有基础内容和社区功能",
        "price": 29.90,
        "currency": "CNY",
        "billing_cycle": "monthly",
        "features": [
          "访问所有付费文章",
          "社区讨论参与",
          "邮件Newsletter订阅"
        ],
        "trial_days": 7,
        "is_active": true,
        "subscriber_count": 156,
        "created_at": "2025-09-01T10:00:00Z",
        "updated_at": "2025-09-15T08:30:00Z"
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 3,
      "pages": 1
    },
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 1.2 创建订阅计划

**接口描述**: 创作者创建新的订阅计划，支持定价策略和试用期设置

```http
POST /api/v1/subscription-plans
```

**权限要求**: 创作者或管理员

**请求体**:
```json
{
  "name": "string(1-100)",
  "description": "string(0-500)",
  "price": "decimal(10,2)",
  "currency": "enum(CNY,USD,EUR)",
  "billing_cycle": "enum(monthly,yearly,lifetime)",
  "features": ["string"],
  "trial_days": "integer(0-30)",
  "is_active": "boolean",
  "metadata": {
    "category": "premium",
    "target_audience": "professionals"
  }
}
```

**请求示例**:
```bash
curl -X POST "/api/v1/subscription-plans" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "name": "高级订阅",
    "description": "完整内容访问权限，包含专属直播和一对一咨询",
    "price": 99.00,
    "currency": "CNY",
    "billing_cycle": "monthly",
    "features": [
      "所有付费内容无限制访问",
      "专属直播课程",
      "月度一对一咨询(30分钟)",
      "专属社群讨论组"
    ],
    "trial_days": 14,
    "is_active": true
  }'
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "plan": {
      "id": 2,
      "name": "高级订阅",
      "description": "完整内容访问权限，包含专属直播和一对一咨询",
      "price": 99.00,
      "currency": "CNY",
      "billing_cycle": "monthly",
      "features": [
        "所有付费内容无限制访问",
        "专属直播课程",
        "月度一对一咨询(30分钟)",
        "专属社群讨论组"
      ],
      "trial_days": 14,
      "is_active": true,
      "subscriber_count": 0,
      "created_at": "2025-09-15T09:25:16Z",
      "updated_at": "2025-09-15T09:25:16Z"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 1.3 更新订阅计划

**接口描述**: 更新现有订阅计划信息，价格变更影响新订阅

```http
PUT /api/v1/subscription-plans/{plan_id}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| plan_id | integer | 是 | 订阅计划ID |

**请求体**: 与创建接口相同，所有字段可选

**重要说明**:
- 价格变更只影响新订阅，现有订阅保持原价
- 停用计划（`is_active: false`）不影响现有订阅
- 功能变更立即对所有订阅者生效

### 1.4 删除订阅计划

**接口描述**: 删除订阅计划，仅允许删除无订阅者的计划

```http
DELETE /api/v1/subscription-plans/{plan_id}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| plan_id | integer | 是 | 订阅计划ID |

**响应示例**:
```json
{
  "success": true,
  "data": {
    "message": "订阅计划已成功删除"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

**错误情况**:
```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "PLAN_HAS_SUBSCRIBERS",
    "message": "无法删除有订阅者的计划",
    "details": {
      "subscriber_count": 15
    }
  }
}
```

---

## 2. 订阅管理接口

### 2.1 创建订阅

**接口描述**: 用户订阅特定创作者的订阅计划

```http
POST /api/v1/subscriptions
```

**请求体**:
```json
{
  "plan_id": "integer",
  "payment_method": "enum(wechat,alipay,stripe)",
  "promo_code": "string(optional)",
  "auto_renew": "boolean",
  "return_url": "string(url,optional)"
}
```

**请求示例**:
```bash
curl -X POST "/api/v1/subscriptions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "plan_id": 1,
    "payment_method": "wechat",
    "auto_renew": true,
    "return_url": "https://app.sharestack.com/subscription/success"
  }'
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "subscription": {
      "id": "sub_abc123def456",
      "plan_id": 1,
      "status": "pending_payment",
      "trial_ends_at": "2025-09-22T09:25:16Z",
      "current_period_start": "2025-09-15T09:25:16Z",
      "current_period_end": "2025-10-15T09:25:16Z",
      "auto_renew": true,
      "created_at": "2025-09-15T09:25:16Z"
    },
    "payment": {
      "id": "pay_xyz789ghi012",
      "amount": 29.90,
      "currency": "CNY",
      "status": "pending",
      "payment_url": "https://api.sharestack.com/payments/wechat/pay_xyz789ghi012",
      "expires_at": "2025-09-15T09:40:16Z"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 2.2 获取用户订阅列表

**接口描述**: 获取当前用户的所有订阅

```http
GET /api/v1/subscriptions
```

**请求参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| status | string | 否 | 状态筛选: `active`, `cancelled`, `expired`, `pending` |
| page | integer | 否 | 页码，默认1 |
| limit | integer | 否 | 每页数量，默认20 |

**响应示例**:
```json
{
  "success": true,
  "data": {
    "subscriptions": [
      {
        "id": "sub_abc123def456",
        "plan": {
          "id": 1,
          "name": "基础订阅",
          "creator": {
            "id": 101,
            "username": "tech_writer",
            "display_name": "科技作家"
          }
        },
        "status": "active",
        "trial_ends_at": null,
        "current_period_start": "2025-09-15T09:25:16Z",
        "current_period_end": "2025-10-15T09:25:16Z",
        "auto_renew": true,
        "next_billing_amount": 29.90,
        "next_billing_date": "2025-10-15T09:25:16Z",
        "created_at": "2025-09-15T09:25:16Z"
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 1,
      "pages": 1
    },
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 2.3 获取订阅详情

**接口描述**: 获取特定订阅的详细信息

```http
GET /api/v1/subscriptions/{subscription_id}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| subscription_id | string | 是 | 订阅ID |

### 2.4 取消订阅自动续费

**接口描述**: 取消订阅的自动续费，订阅在当前周期结束后停止

```http
PUT /api/v1/subscriptions/{subscription_id}/cancel
```

**请求体**:
```json
{
  "reason": "enum(too_expensive,not_satisfied,found_alternative,other)",
  "feedback": "string(optional,0-500)"
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "subscription": {
      "id": "sub_abc123def456",
      "status": "cancelled",
      "auto_renew": false,
      "cancelled_at": "2025-09-15T09:25:16Z",
      "access_until": "2025-10-15T09:25:16Z"
    },
    "message": "订阅已取消，您可以继续访问内容直到 2025-10-15"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 2.5 订阅升级降级

**接口描述**: 更改订阅计划，支持即时升级和周期结束时降级

```http
PUT /api/v1/subscriptions/{subscription_id}/change-plan
```

**请求体**:
```json
{
  "new_plan_id": "integer",
  "change_timing": "enum(immediate,next_cycle)",
  "prorate": "boolean"
}
```

**升级示例** (immediate):
```bash
curl -X PUT "/api/v1/subscriptions/sub_abc123def456/change-plan" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "new_plan_id": 2,
    "change_timing": "immediate",
    "prorate": true
  }'
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "subscription": {
      "id": "sub_abc123def456",
      "plan_id": 2,
      "status": "active",
      "changed_at": "2025-09-15T09:25:16Z"
    },
    "payment": {
      "id": "pay_upgrade_123",
      "amount": 23.27,
      "description": "升级到高级订阅的差额费用（按比例计算）",
      "payment_url": "https://api.sharestack.com/payments/wechat/pay_upgrade_123"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

---

## 3. 支付处理接口

### 3.1 创建支付订单

**接口描述**: 创建支付订单，支持多种支付方式

```http
POST /api/v1/payments/create
```

**请求体**:
```json
{
  "amount": "decimal(10,2)",
  "currency": "enum(CNY,USD,EUR)",
  "payment_method": "enum(wechat,alipay,stripe)",
  "order_type": "enum(subscription,one_time)",
  "order_id": "string",
  "description": "string",
  "return_url": "string(url,optional)",
  "metadata": {
    "subscription_id": "string",
    "user_id": "integer"
  },
  "idempotency_key": "string"
}
```

**微信支付示例**:
```bash
curl -X POST "/api/v1/payments/create" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "amount": 29.90,
    "currency": "CNY",
    "payment_method": "wechat",
    "order_type": "subscription",
    "order_id": "sub_abc123def456",
    "description": "基础订阅 - 月付",
    "return_url": "https://app.sharestack.com/payment/success",
    "idempotency_key": "idem_user123_20250915_001"
  }'
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "payment": {
      "id": "pay_xyz789ghi012",
      "amount": 29.90,
      "currency": "CNY",
      "status": "pending",
      "payment_method": "wechat",
      "expires_at": "2025-09-15T09:40:16Z",
      "created_at": "2025-09-15T09:25:16Z"
    },
    "payment_info": {
      "wechat": {
        "prepay_id": "wx15092509251612345678",
        "qr_code_url": "https://pay.weixin.qq.com/qr/abc123def456",
        "deep_link": "weixin://wxpay/bizpayurl?pr=abc123def456"
      }
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

**Stripe支付响应**:
```json
{
  "payment_info": {
    "stripe": {
      "client_secret": "pi_1ABC123DEF456_secret_xyz789",
      "publishable_key": "pk_test_123456789",
      "payment_intent_id": "pi_1ABC123DEF456"
    }
  }
}
```

### 3.2 获取支付状态

**接口描述**: 查询支付订单的当前状态

```http
GET /api/v1/payments/{payment_id}
```

**路径参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| payment_id | string | 是 | 支付订单ID |

**响应示例**:
```json
{
  "success": true,
  "data": {
    "payment": {
      "id": "pay_xyz789ghi012",
      "amount": 29.90,
      "currency": "CNY",
      "status": "completed",
      "payment_method": "wechat",
      "transaction_id": "wx_trans_abc123def456",
      "paid_at": "2025-09-15T09:28:45Z",
      "created_at": "2025-09-15T09:25:16Z",
      "metadata": {
        "subscription_id": "sub_abc123def456",
        "user_id": 123
      }
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:30:00Z"
  }
}
```

### 3.3 确认支付

**接口描述**: 手动确认支付完成（主要用于Stripe 3D验证）

```http
POST /api/v1/payments/{payment_id}/confirm
```

**请求体**:
```json
{
  "payment_intent_id": "string(stripe专用)",
  "payment_method_id": "string(stripe专用)"
}
```

### 3.4 微信支付回调

**接口描述**: 微信支付异步通知接口

```http
POST /api/v1/payments/webhooks/wechat
```

**请求头**:
```
Wechatpay-Serial: 微信支付平台证书序列号
Wechatpay-Signature: 微信支付签名
Wechatpay-Timestamp: 时间戳
Wechatpay-Nonce: 随机字符串
```

**回调处理流程**:
1. 验证微信支付签名
2. 解密回调数据
3. 更新支付状态
4. 激活相关订阅
5. 返回成功响应

**响应示例**:
```json
{
  "code": "SUCCESS",
  "message": "成功"
}
```

### 3.5 支付宝回调

**接口描述**: 支付宝异步通知接口

```http
POST /api/v1/payments/webhooks/alipay
```

**回调验证**:
- 验证支付宝RSA签名
- 检查商户订单号和支付金额
- 确认交易状态

### 3.6 Stripe Webhook回调

**接口描述**: Stripe事件回调接口

```http
POST /api/v1/payments/webhooks/stripe
```

**请求头**:
```
Stripe-Signature: webhook签名
```

**处理事件类型**:
- `payment_intent.succeeded`: 支付成功
- `payment_intent.payment_failed`: 支付失败
- `invoice.payment_succeeded`: 订阅续费成功
- `customer.subscription.deleted`: 订阅取消

---

## 4. 财务管理接口

### 4.1 获取收入统计

**接口描述**: 创作者查看收入统计数据

```http
GET /api/v1/earnings
```

**请求参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| period | string | 否 | 统计周期: `today`, `week`, `month`, `quarter`, `year` |
| start_date | date | 否 | 开始日期 (YYYY-MM-DD) |
| end_date | date | 否 | 结束日期 (YYYY-MM-DD) |
| currency | string | 否 | 货币类型筛选 |

**请求示例**:
```bash
curl -X GET "/api/v1/earnings?period=month&currency=CNY" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "summary": {
      "total_revenue": 15680.50,
      "net_earnings": 13826.84,
      "platform_fee": 1568.05,
      "tax_withheld": 285.61,
      "subscriber_count": 89,
      "new_subscribers": 23,
      "cancelled_subscribers": 5
    },
    "breakdown": {
      "subscription_revenue": 14580.00,
      "one_time_purchases": 1100.50,
      "tips": 0.00
    },
    "currency": "CNY",
    "period": {
      "start": "2025-09-01",
      "end": "2025-09-30"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 4.2 获取交易记录

**接口描述**: 查看详细的交易明细

```http
GET /api/v1/earnings/transactions
```

**请求参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| type | string | 否 | 交易类型: `subscription`, `one_time`, `refund`, `tip` |
| status | string | 否 | 状态: `completed`, `pending`, `failed`, `refunded` |
| page | integer | 否 | 页码，默认1 |
| limit | integer | 否 | 每页数量，默认50 |

**响应示例**:
```json
{
  "success": true,
  "data": {
    "transactions": [
      {
        "id": "txn_abc123def456",
        "type": "subscription",
        "status": "completed",
        "amount": 29.90,
        "platform_fee": 2.99,
        "net_amount": 26.91,
        "currency": "CNY",
        "payment_method": "wechat",
        "subscriber": {
          "id": 456,
          "username": "reader123"
        },
        "subscription": {
          "id": "sub_abc123def456",
          "plan_name": "基础订阅"
        },
        "processed_at": "2025-09-15T09:25:16Z"
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 50,
      "total": 156,
      "pages": 4
    },
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 4.3 申请提现

**接口描述**: 创作者申请提现

```http
POST /api/v1/withdrawals
```

**请求体**:
```json
{
  "amount": "decimal(10,2)",
  "currency": "enum(CNY,USD)",
  "bank_account": {
    "bank_name": "string",
    "account_number": "string",
    "account_holder": "string",
    "routing_number": "string(optional)"
  },
  "tax_declaration": {
    "tax_resident_country": "string",
    "tax_id": "string(optional)",
    "declaration_signed": "boolean"
  }
}
```

**请求示例**:
```bash
curl -X POST "/api/v1/withdrawals" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -d '{
    "amount": 5000.00,
    "currency": "CNY",
    "bank_account": {
      "bank_name": "中国工商银行",
      "account_number": "6222021234567890123",
      "account_holder": "张三"
    },
    "tax_declaration": {
      "tax_resident_country": "CN",
      "declaration_signed": true
    }
  }'
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "withdrawal": {
      "id": "wd_xyz789abc123",
      "amount": 5000.00,
      "currency": "CNY",
      "status": "pending_review",
      "processing_fee": 10.00,
      "net_amount": 4990.00,
      "estimated_arrival": "2025-09-18T00:00:00Z",
      "submitted_at": "2025-09-15T09:25:16Z"
    },
    "message": "提现申请已提交，预计3个工作日内到账"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 4.4 获取提现记录

**接口描述**: 查看提现历史记录

```http
GET /api/v1/withdrawals
```

**请求参数**:
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| status | string | 否 | 状态筛选: `pending_review`, `processing`, `completed`, `failed` |
| page | integer | 否 | 页码，默认1 |
| limit | integer | 否 | 每页数量，默认20 |

### 4.5 获取税务信息

**接口描述**: 获取用户的税务信息和设置

```http
GET /api/v1/tax-info
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "tax_info": {
      "tax_resident_country": "CN",
      "tax_id": "123456789012345678",
      "tax_id_type": "personal_id",
      "withholding_rate": 0.20,
      "vat_number": null,
      "tax_treaty_benefits": false,
      "updated_at": "2025-09-01T10:00:00Z"
    },
    "tax_documents": [
      {
        "year": 2025,
        "document_type": "annual_statement",
        "download_url": "https://documents.sharestack.com/tax/2025_annual_statement.pdf",
        "generated_at": "2025-01-15T10:00:00Z"
      }
    ]
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T09:25:16Z"
  }
}
```

### 4.6 更新税务信息

**接口描述**: 更新税务信息和设置

```http
PUT /api/v1/tax-info
```

**请求体**:
```json
{
  "tax_resident_country": "string(ISO 3166-1 alpha-2)",
  "tax_id": "string",
  "tax_id_type": "enum(personal_id,tax_number,passport)",
  "vat_number": "string(optional)",
  "tax_treaty_benefits": "boolean",
  "supporting_documents": ["string(document_ids)"]
}
```

---

## 5. 支付渠道集成方案

### 5.1 微信支付集成

#### 支付流程
1. **创建支付订单** → 调用微信支付统一下单API
2. **展示支付二维码** → 用户扫码或使用深度链接
3. **支付完成** → 微信发送异步通知
4. **订阅激活** → 更新订阅状态

#### 配置要求
```json
{
  "merchant_id": "微信支付商户号",
  "app_id": "微信AppID",
  "api_key": "微信支付API密钥",
  "cert_path": "API证书文件路径",
  "notify_url": "https://api.sharestack.com/api/v1/payments/webhooks/wechat"
}
```

#### 安全措施
- 使用微信支付平台证书验证回调签名
- 敏感数据AES-256-GCM加密传输
- 实现幂等性处理，防止重复处理回调

### 5.2 支付宝集成

#### 支付流程
1. **创建支付订单** → 调用支付宝手机网站支付API
2. **跳转支付页面** → 用户完成支付宝认证
3. **同步返回** → 支付宝同步返回支付结果
4. **异步通知** → 支付宝发送异步通知确认

#### 配置要求
```json
{
  "app_id": "支付宝应用ID",
  "private_key": "应用私钥",
  "public_key": "支付宝公钥",
  "sign_type": "RSA2",
  "notify_url": "https://api.sharestack.com/api/v1/payments/webhooks/alipay"
}
```

### 5.3 Stripe国际支付

#### 支付流程
1. **创建PaymentIntent** → Stripe服务端创建支付意图
2. **客户端确认** → 前端使用Stripe.js确认支付
3. **3D验证** → 支持3D Secure认证流程
4. **Webhook通知** → Stripe发送事件通知

#### 配置要求
```json
{
  "publishable_key": "pk_live_...",
  "secret_key": "sk_live_...",
  "webhook_secret": "whsec_...",
  "webhook_url": "https://api.sharestack.com/api/v1/payments/webhooks/stripe"
}
```

#### 支持的支付方式
- **信用卡**: Visa, Mastercard, American Express
- **数字钱包**: Apple Pay, Google Pay
- **本地支付**: Bancontact, iDEAL, SEPA Debit
- **后付款**: Klarna, Afterpay

### 5.4 统一支付接口设计

#### 抽象支付处理器
```python
class PaymentProcessor:
    def create_payment(self, amount, currency, method, metadata):
        """创建支付订单，返回支付信息"""
        pass

    def verify_webhook(self, headers, body):
        """验证webhook签名和数据"""
        pass

    def process_webhook(self, event_data):
        """处理webhook事件"""
        pass
```

#### 支付状态映射
| 平台状态 | 统一状态 | 描述 |
|----------|----------|------|
| 微信: NOTPAY | pending | 等待支付 |
| 支付宝: WAIT_BUYER_PAY | pending | 等待支付 |
| Stripe: requires_payment_method | pending | 等待支付 |
| 微信: SUCCESS | completed | 支付成功 |
| 支付宝: TRADE_SUCCESS | completed | 支付成功 |
| Stripe: succeeded | completed | 支付成功 |

---

## 6. 错误码和异常处理

### 6.1 支付相关错误码

| 错误码 | HTTP状态 | 描述 | 解决方案 |
|--------|----------|------|----------|
| PAYMENT_AMOUNT_INVALID | 400 | 支付金额无效 | 检查金额格式和范围 |
| PAYMENT_METHOD_UNSUPPORTED | 400 | 不支持的支付方式 | 使用支持的支付渠道 |
| PAYMENT_EXPIRED | 410 | 支付订单已过期 | 重新创建支付订单 |
| PAYMENT_ALREADY_PAID | 409 | 订单已支付 | 检查订单状态 |
| PAYMENT_FAILED | 402 | 支付失败 | 重试支付或更换支付方式 |
| INSUFFICIENT_BALANCE | 402 | 余额不足 | 充值或更换支付方式 |
| PAYMENT_RISK_BLOCKED | 403 | 风控拦截 | 联系客服或实名认证 |

### 6.2 订阅相关错误码

| 错误码 | HTTP状态 | 描述 | 解决方案 |
|--------|----------|------|----------|
| SUBSCRIPTION_NOT_FOUND | 404 | 订阅不存在 | 检查订阅ID |
| SUBSCRIPTION_ALREADY_EXISTS | 409 | 已存在相同订阅 | 查看现有订阅状态 |
| SUBSCRIPTION_CANCELLED | 410 | 订阅已取消 | 重新订阅 |
| PLAN_NOT_ACTIVE | 400 | 订阅计划未激活 | 选择其他计划 |
| PLAN_NOT_FOUND | 404 | 订阅计划不存在 | 检查计划ID |

### 6.3 财务相关错误码

| 错误码 | HTTP状态 | 描述 | 解决方案 |
|--------|----------|------|----------|
| INSUFFICIENT_EARNINGS | 400 | 收入余额不足 | 等待更多收入 |
| WITHDRAWAL_LIMIT_EXCEEDED | 429 | 超出提现限额 | 降低提现金额 |
| BANK_ACCOUNT_INVALID | 400 | 银行账户信息无效 | 检查账户信息 |
| TAX_INFO_INCOMPLETE | 400 | 税务信息不完整 | 完善税务信息 |
| WITHDRAWAL_PENDING | 409 | 存在待处理提现 | 等待处理完成 |

### 6.4 安全相关错误码

| 错误码 | HTTP状态 | 描述 | 解决方案 |
|--------|----------|------|----------|
| SIGNATURE_INVALID | 401 | 签名验证失败 | 检查签名算法和密钥 |
| WEBHOOK_REPLAY_ATTACK | 400 | 疑似重放攻击 | 检查时间戳和随机数 |
| RATE_LIMIT_EXCEEDED | 429 | 请求频率超限 | 降低请求频率 |
| IP_BLOCKED | 403 | IP地址被拦截 | 联系技术支持 |

### 6.5 错误处理最佳实践

#### 客户端重试策略
```javascript
const retryConfig = {
  // 可重试的错误类型
  retryableErrors: ['PAYMENT_NETWORK_ERROR', 'PAYMENT_TIMEOUT'],
  // 重试次数
  maxRetries: 3,
  // 退避策略：指数退避 + 抖动
  backoffFactor: 2,
  baseDelay: 1000,
  maxDelay: 30000
};
```

#### 优雅降级
- 支付失败时提供替代支付方式
- 网络异常时显示离线提示
- 服务不可用时启用维护模式

---

## 7. 性能和监控

### 7.1 性能指标

#### 接口响应时间基准
| 接口类型 | P50 | P95 | P99 |
|----------|-----|-----|-----|
| 订阅计划查询 | <100ms | <200ms | <500ms |
| 创建支付订单 | <300ms | <800ms | <1500ms |
| 支付状态查询 | <50ms | <100ms | <200ms |
| Webhook处理 | <200ms | <500ms | <1000ms |
| 收入统计查询 | <500ms | <1000ms | <2000ms |

#### 并发处理能力
- 支付创建：1000 req/s
- 状态查询：5000 req/s
- Webhook处理：500 req/s

### 7.2 监控指标

#### 业务指标监控
```json
{
  "payment_success_rate": "支付成功率 >95%",
  "webhook_processing_rate": "Webhook处理成功率 >99%",
  "subscription_activation_rate": "订阅激活率 >98%",
  "withdrawal_processing_time": "提现处理时间 <3天"
}
```

#### 技术指标监控
```json
{
  "api_availability": "接口可用性 >99.9%",
  "response_time_p95": "P95响应时间 <500ms",
  "error_rate": "错误率 <1%",
  "database_connection_pool": "数据库连接池使用率 <80%"
}
```

### 7.3 告警配置

#### 关键告警
- 支付成功率低于90%（立即）
- Webhook处理失败超过100次/分钟（立即）
- 提现处理超时超过5天（1小时内）
- API错误率超过5%（5分钟内）

#### 预警监控
- 数据库连接数接近上限
- Redis内存使用率超过80%
- 支付渠道响应时间异常增长

---

## 8. 财务合规和税务处理

### 8.1 税务合规要求

#### 中国大陆税务处理
- **个人所得税**: 收入达到起征点时自动代扣代缴
- **增值税**: 企业创作者需要开具发票
- **预扣预缴**: 平台按规定进行预扣预缴申报

#### 国际税务处理（Stripe）
- **美国税务**: 支持W-8BEN和W-9税务表格
- **欧盟VAT**: 支持MOSS一站式服务
- **税务条约**: 自动适用双边税收协定优惠

### 8.2 合规文档管理

#### 自动生成文档
```http
GET /api/v1/tax-documents/{year}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "documents": [
      {
        "type": "annual_income_statement",
        "year": 2025,
        "total_income": 156800.50,
        "tax_withheld": 31360.10,
        "download_url": "https://documents.sharestack.com/tax/2025_income.pdf",
        "generated_at": "2025-01-15T10:00:00Z"
      },
      {
        "type": "monthly_transaction_report",
        "period": "2025-09",
        "download_url": "https://documents.sharestack.com/tax/2025_09_transactions.csv"
      }
    ]
  }
}
```

### 8.3 反洗钱（AML）合规

#### 身份验证等级
| 等级 | 提现限额 | 验证要求 |
|------|----------|----------|
| L1-基础 | ¥10,000/月 | 手机号+邮箱验证 |
| L2-标准 | ¥50,000/月 | 身份证实名认证 |
| L3-增强 | ¥200,000/月 | 银行卡四要素验证 |
| L4-企业 | 不限额 | 企业营业执照+法人认证 |

#### 可疑交易监控
- 单日大额交易（>¥50,000）
- 频繁小额交易（>100次/日）
- 异地IP登录后立即提现
- 关联账户资金集中流动

---

## 9. 安全最佳实践

### 9.1 API安全设计

#### 访问控制
```javascript
// JWT Token包含的权限信息
{
  "user_id": 123,
  "role": "creator",
  "permissions": [
    "subscription:read",
    "subscription:write",
    "payment:read",
    "earnings:read"
  ],
  "subscription_access": ["sub_abc123", "sub_def456"]
}
```

#### 敏感数据保护
- **PII加密**: 银行账号、身份证号使用AES-256加密存储
- **日志脱敏**: 支付相关日志自动脱敏处理
- **数据最小化**: 仅收集和处理必要的用户数据

### 9.2 支付安全防护

#### 防重复支付
```python
class IdempotencyChecker:
    def __init__(self, redis_client):
        self.redis = redis_client

    def check_and_set(self, key, ttl=900):  # 15分钟
        if self.redis.exists(key):
            return False  # 重复请求
        self.redis.setex(key, ttl, "processing")
        return True
```

#### 风控规则引擎
```json
{
  "rules": [
    {
      "name": "large_amount_check",
      "condition": "amount > 10000 AND user.kyc_level < 3",
      "action": "require_additional_verification"
    },
    {
      "name": "velocity_check",
      "condition": "payment_count_24h > 50",
      "action": "rate_limit"
    },
    {
      "name": "device_check",
      "condition": "device_fingerprint NOT IN user.known_devices",
      "action": "require_2fa"
    }
  ]
}
```

### 9.3 数据隐私保护

#### GDPR合规
- **数据可携带**: 支持用户数据导出
- **被遗忘权**: 支持账户删除和数据清理
- **数据最小化**: 仅收集业务必需数据
- **同意管理**: 明确的数据处理同意机制

#### 数据匿名化
```python
def anonymize_transaction_data(transaction):
    """交易数据匿名化处理"""
    return {
        "amount": transaction.amount,
        "currency": transaction.currency,
        "timestamp": transaction.created_at,
        "user_id_hash": hash_user_id(transaction.user_id),
        # 移除所有PII信息
    }
```

---

## 10. 开发和测试指南

### 10.1 本地开发环境

#### 环境变量配置
```env
# 数据库配置
DATABASE_URL=mysql://user:pass@localhost:3306/sharestack
REDIS_URL=redis://localhost:6379/0

# 支付配置
WECHAT_MERCHANT_ID=your_merchant_id
WECHAT_API_KEY=your_api_key
ALIPAY_APP_ID=your_app_id
STRIPE_SECRET_KEY=sk_test_...

# 安全配置
JWT_SECRET=your_jwt_secret
ENCRYPTION_KEY=your_encryption_key
```

#### 本地测试支付
```bash
# 启动支付回调测试服务器
ngrok http 8000

# 更新webhook URL
export WEBHOOK_BASE_URL=https://abc123.ngrok.io
```

### 10.2 测试数据和Mock

#### 测试支付账号
```json
{
  "wechat_test": {
    "merchant_id": "1900000001",
    "api_key": "test_key_32_characters_long"
  },
  "stripe_test": {
    "publishable_key": "pk_test_...",
    "secret_key": "sk_test_..."
  }
}
```

#### Mock支付响应
```python
@pytest.fixture
def mock_wechat_payment():
    return {
        "prepay_id": "wx15092509251612345678",
        "code_url": "weixin://wxpay/bizpayurl?pr=test123",
        "return_code": "SUCCESS",
        "result_code": "SUCCESS"
    }
```

### 10.3 API测试套件

#### Postman集合示例
```json
{
  "name": "ShareStack Subscription Payment API",
  "variables": [
    {
      "key": "base_url",
      "value": "https://api.sharestack.com/api/v1"
    },
    {
      "key": "auth_token",
      "value": "{{access_token}}"
    }
  ],
  "requests": [
    {
      "name": "Create Subscription Plan",
      "method": "POST",
      "url": "{{base_url}}/subscription-plans",
      "headers": {
        "Authorization": "Bearer {{auth_token}}"
      }
    }
  ]
}
```

#### 自动化测试脚本
```python
class TestSubscriptionPaymentAPI:
    def test_create_subscription_flow(self):
        # 1. 创建订阅计划
        plan = self.create_test_plan()

        # 2. 创建订阅
        subscription = self.create_subscription(plan.id)

        # 3. 创建支付
        payment = self.create_payment(subscription.id)

        # 4. 模拟支付成功
        self.simulate_payment_success(payment.id)

        # 5. 验证订阅激活
        assert subscription.status == "active"
```

---

## 附录

### A. 支付状态机

```mermaid
stateDiagram-v2
    [*] --> pending: 创建支付
    pending --> processing: 用户发起支付
    pending --> expired: 超时未支付
    processing --> completed: 支付成功
    processing --> failed: 支付失败
    completed --> refunded: 申请退款
    failed --> pending: 重新支付
    expired --> [*]
    refunded --> [*]
```

### B. 订阅生命周期

```mermaid
stateDiagram-v2
    [*] --> pending_payment: 创建订阅
    pending_payment --> trialing: 支付成功(有试用期)
    pending_payment --> active: 支付成功(无试用期)
    trialing --> active: 试用期结束
    active --> cancelled: 取消自动续费
    active --> past_due: 续费失败
    cancelled --> active: 重新激活
    past_due --> active: 补缴成功
    past_due --> cancelled: 超期未缴
    cancelled --> [*]: 访问期结束
```

### C. Webhook事件列表

| 事件名称 | 触发时机 | 包含数据 |
|----------|----------|----------|
| `payment.created` | 支付订单创建 | payment对象 |
| `payment.completed` | 支付成功 | payment对象 + transaction_id |
| `payment.failed` | 支付失败 | payment对象 + error_details |
| `subscription.created` | 订阅创建 | subscription对象 |
| `subscription.activated` | 订阅激活 | subscription对象 |
| `subscription.cancelled` | 订阅取消 | subscription对象 + cancellation_reason |
| `subscription.renewed` | 订阅续费 | subscription对象 + payment对象 |
| `withdrawal.requested` | 提现申请 | withdrawal对象 |
| `withdrawal.completed` | 提现完成 | withdrawal对象 + bank_details |

### D. 货币和汇率处理

#### 支持的货币列表
```json
{
  "supported_currencies": [
    {"code": "CNY", "name": "人民币", "symbol": "¥", "decimal_places": 2},
    {"code": "USD", "name": "美元", "symbol": "$", "decimal_places": 2},
    {"code": "EUR", "name": "欧元", "symbol": "€", "decimal_places": 2},
    {"code": "GBP", "name": "英镑", "symbol": "£", "decimal_places": 2},
    {"code": "JPY", "name": "日元", "symbol": "¥", "decimal_places": 0}
  ]
}
```

#### 汇率API
```http
GET /api/v1/exchange-rates?from=USD&to=CNY
```

### E. 速率限制配置

| 接口类型 | 限制规则 | 说明 |
|----------|----------|------|
| 支付创建 | 100/用户/小时 | 防止恶意创建订单 |
| 支付查询 | 1000/用户/小时 | 允许频繁状态检查 |
| 订阅管理 | 50/用户/小时 | 防止频繁变更 |
| Webhook | 无限制 | 支付平台回调 |
| 提现申请 | 10/用户/天 | 防止频繁提现 |

---

**文档版本**: v1.0.0
**创建时间**: 2025-09-15
**最后更新**: 2025-09-15
**维护团队**: ShareStack 后端开发团队

*本文档提供ShareStack知识付费平台订阅支付功能的完整API规范，涵盖订阅管理、多渠道支付集成、财务统计和安全合规要求。所有接口设计遵循RESTful规范和OpenAPI 3.0标准，确保开发团队能够高效实现和维护支付系统。*