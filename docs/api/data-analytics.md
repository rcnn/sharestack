# ShareStack 数据分析 API 文档

## 概述

ShareStack 数据分析模块提供全面的数据洞察功能，包括创作者分析、平台监控和通知系统。本文档遵循 OpenAPI 3.0 规范，为所有数据分析相关接口提供详细的规范说明。

### 基础信息

- **API 版本**: v1
- **基础 URL**: `https://api.sharestack.com/api/v1`
- **认证方式**: Bearer Token (JWT)
- **响应格式**: JSON

### 数据聚合特性

- **时间序列数据**: 支持小时、天、周、月多种粒度聚合
- **实时数据**: 关键指标实时更新，延迟 < 5秒
- **数据导出**: 支持 JSON、CSV、Excel 格式导出
- **数据可视化**: 提供标准化图表数据格式

### 通用响应格式

#### 成功响应
```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "123ms",
    "cache_hit": false
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
| DATA_NOT_FOUND | 404 | 数据不存在 |
| RATE_LIMIT_EXCEEDED | 429 | 请求频率超限 |
| PROCESSING_ERROR | 422 | 数据处理失败 |
| INTERNAL_ERROR | 500 | 服务器内部错误 |

---

## 1. 创作者分析接口

### 1.1 数据概览

**接口路径**: `GET /analytics/overview`

**描述**: 获取创作者个人数据概览，包括核心指标和趋势数据。

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| timezone | string | 否 | 时区，默认UTC+8 |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "overview": {
      "total_articles": 156,
      "total_subscribers": 2847,
      "total_revenue": 45632.50,
      "total_views": 128450,
      "avg_engagement_rate": 0.073
    },
    "current_period": {
      "articles_published": 8,
      "new_subscribers": 234,
      "revenue": 3456.80,
      "views": 12450,
      "engagement_rate": 0.081
    },
    "trends": {
      "subscribers_growth": 0.089,
      "revenue_growth": 0.124,
      "views_growth": 0.056,
      "engagement_growth": 0.092
    },
    "top_articles": [
      {
        "article_id": 789,
        "title": "深入理解机器学习算法",
        "views": 2340,
        "likes": 156,
        "comments": 23,
        "revenue": 567.80,
        "published_at": "2025-09-10T08:00:00Z"
      }
    ]
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "89ms",
    "cache_hit": true
  }
}
```

### 1.2 文章分析

**接口路径**: `GET /analytics/articles`

**描述**: 获取文章性能分析数据，包括阅读量、互动数据和转化指标。

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| article_id | integer | 否 | 特定文章ID，不传则返回所有文章 |
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| metric | string | 否 | 指标类型：views, engagement, revenue，默认all |
| sort_by | string | 否 | 排序字段：views, likes, revenue, published_at |
| sort_order | string | 否 | 排序方向：asc, desc，默认desc |
| page | integer | 否 | 页码，默认1 |
| limit | integer | 否 | 每页数量，默认20，最大100 |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "articles": [
      {
        "article_id": 789,
        "title": "深入理解机器学习算法",
        "slug": "deep-learning-algorithms",
        "published_at": "2025-09-10T08:00:00Z",
        "metrics": {
          "views": 2340,
          "unique_visitors": 1987,
          "avg_read_time": 420,
          "completion_rate": 0.67,
          "likes": 156,
          "comments": 23,
          "shares": 45,
          "bookmarks": 89,
          "conversion_rate": 0.078,
          "revenue": 567.80
        },
        "time_series": {
          "daily_views": [120, 156, 189, 234, 278, 301, 267],
          "daily_revenue": [23.4, 34.5, 45.6, 67.8, 89.1, 102.3, 87.6]
        },
        "traffic_sources": {
          "direct": 0.34,
          "search": 0.28,
          "social": 0.21,
          "referral": 0.17
        }
      }
    ],
    "summary": {
      "total_articles": 156,
      "total_views": 45620,
      "total_revenue": 8934.56,
      "avg_engagement_rate": 0.073
    },
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "pages": 8
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "145ms",
    "cache_hit": false
  }
}
```

### 1.3 收入分析

**接口路径**: `GET /analytics/revenue`

**描述**: 获取创作者收入分析数据，包括订阅收入、单篇收入和趋势预测。

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| granularity | string | 否 | 时间粒度：hour, day, week, month，默认day |
| currency | string | 否 | 货币单位：CNY, USD，默认CNY |
| include_prediction | boolean | 否 | 是否包含预测数据，默认false |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "revenue_summary": {
      "total_revenue": 45632.50,
      "subscription_revenue": 38456.20,
      "single_article_revenue": 7176.30,
      "avg_daily_revenue": 1521.08,
      "revenue_growth_rate": 0.124
    },
    "revenue_breakdown": {
      "by_subscription_plan": [
        {
          "plan_name": "月度订阅",
          "subscribers_count": 2340,
          "revenue": 23400.00,
          "percentage": 0.513
        },
        {
          "plan_name": "年度订阅",
          "subscribers_count": 456,
          "revenue": 15056.20,
          "percentage": 0.330
        }
      ],
      "by_article_type": [
        {
          "type": "付费文章",
          "articles_count": 23,
          "revenue": 7176.30,
          "avg_revenue_per_article": 312.01
        }
      ]
    },
    "time_series": {
      "labels": ["2025-09-01", "2025-09-02", "2025-09-03"],
      "subscription_revenue": [156.80, 178.90, 189.50],
      "article_revenue": [45.60, 67.80, 89.10],
      "total_revenue": [202.40, 246.70, 278.60]
    },
    "prediction": {
      "enabled": false,
      "next_30_days_estimate": null,
      "confidence_interval": null
    },
    "top_earning_articles": [
      {
        "article_id": 789,
        "title": "深入理解机器学习算法",
        "revenue": 567.80,
        "views": 2340,
        "conversion_rate": 0.078
      }
    ]
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "167ms",
    "cache_hit": false
  }
}
```

### 1.4 受众分析

**接口路径**: `GET /analytics/audience`

**描述**: 获取受众行为分析数据，包括人口统计、行为模式和兴趣分析。

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| segment | string | 否 | 受众细分：all, subscribers, readers，默认all |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "audience_overview": {
      "total_followers": 2847,
      "active_subscribers": 2340,
      "avg_engagement_rate": 0.073,
      "churn_rate": 0.034,
      "ltv": 156.78
    },
    "demographics": {
      "age_distribution": {
        "18-24": 0.12,
        "25-34": 0.43,
        "35-44": 0.31,
        "45-54": 0.10,
        "55+": 0.04
      },
      "gender_distribution": {
        "male": 0.64,
        "female": 0.34,
        "other": 0.02
      },
      "geographic_distribution": {
        "北京": 0.18,
        "上海": 0.16,
        "深圳": 0.12,
        "广州": 0.09,
        "杭州": 0.08,
        "其他": 0.37
      }
    },
    "behavior_patterns": {
      "reading_habits": {
        "avg_articles_per_month": 8.5,
        "avg_reading_time": 420,
        "peak_reading_hours": [9, 14, 20, 22],
        "preferred_article_length": "中等长度(1000-3000字)"
      },
      "engagement_patterns": {
        "like_rate": 0.067,
        "comment_rate": 0.034,
        "share_rate": 0.021,
        "bookmark_rate": 0.045
      }
    },
    "content_preferences": {
      "popular_categories": [
        {
          "category": "技术教程",
          "interest_score": 0.87,
          "engagement_rate": 0.094
        },
        {
          "category": "行业分析",
          "interest_score": 0.76,
          "engagement_rate": 0.081
        }
      ]
    },
    "subscriber_lifecycle": {
      "new_subscribers": 234,
      "retained_subscribers": 2106,
      "churned_subscribers": 78,
      "retention_rate": 0.966
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "234ms",
    "cache_hit": true
  }
}
```

### 1.5 流量分析

**接口路径**: `GET /analytics/traffic`

**描述**: 获取流量来源分析数据，包括渠道分析、设备统计和用户行为路径。

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| metric | string | 否 | 指标类型：sessions, users, pageviews，默认sessions |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "traffic_overview": {
      "total_sessions": 15678,
      "unique_visitors": 12340,
      "page_views": 45629,
      "avg_session_duration": 420,
      "bounce_rate": 0.34
    },
    "traffic_sources": {
      "channels": [
        {
          "source": "直接访问",
          "sessions": 5320,
          "percentage": 0.339,
          "avg_session_duration": 480,
          "bounce_rate": 0.28
        },
        {
          "source": "搜索引擎",
          "sessions": 4389,
          "percentage": 0.280,
          "avg_session_duration": 390,
          "bounce_rate": 0.35
        },
        {
          "source": "社交媒体",
          "sessions": 3292,
          "percentage": 0.210,
          "avg_session_duration": 360,
          "bounce_rate": 0.42
        },
        {
          "source": "外部链接",
          "sessions": 2677,
          "percentage": 0.171,
          "avg_session_duration": 420,
          "bounce_rate": 0.31
        }
      ],
      "top_referrers": [
        {
          "domain": "baidu.com",
          "sessions": 2156,
          "conversion_rate": 0.067
        },
        {
          "domain": "weibo.com",
          "sessions": 1890,
          "conversion_rate": 0.045
        }
      ]
    },
    "device_analytics": {
      "device_types": {
        "desktop": 0.45,
        "mobile": 0.52,
        "tablet": 0.03
      },
      "operating_systems": {
        "Windows": 0.42,
        "Android": 0.31,
        "iOS": 0.21,
        "macOS": 0.04,
        "Linux": 0.02
      },
      "browsers": {
        "Chrome": 0.67,
        "Safari": 0.18,
        "Firefox": 0.08,
        "Edge": 0.05,
        "Other": 0.02
      }
    },
    "user_flow": {
      "top_landing_pages": [
        {
          "path": "/articles/789",
          "sessions": 2340,
          "bounce_rate": 0.23
        }
      ],
      "top_exit_pages": [
        {
          "path": "/profile",
          "exit_rate": 0.45
        }
      ]
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "189ms",
    "cache_hit": false
  }
}
```

---

## 2. 平台数据监控接口

### 2.1 平台数据概览

**接口路径**: `GET /admin/analytics/platform`

**描述**: 获取平台整体数据概览，包括用户增长、内容统计和收入指标。

**权限要求**: 需要管理员权限

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| include_trends | boolean | 否 | 是否包含趋势数据，默认true |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "platform_overview": {
      "total_users": 125680,
      "total_creators": 8945,
      "total_articles": 45672,
      "total_subscriptions": 23451,
      "platform_revenue": 1234567.89,
      "monthly_active_users": 89456
    },
    "growth_metrics": {
      "user_growth_rate": 0.156,
      "creator_growth_rate": 0.234,
      "content_growth_rate": 0.189,
      "revenue_growth_rate": 0.267
    },
    "engagement_metrics": {
      "avg_session_duration": 445,
      "avg_pages_per_session": 3.4,
      "daily_active_users": 34567,
      "user_retention_rate": 0.78,
      "content_interaction_rate": 0.067
    },
    "content_statistics": {
      "articles_published_today": 234,
      "avg_articles_per_creator": 5.1,
      "most_popular_categories": [
        {
          "category": "技术教程",
          "article_count": 8945,
          "engagement_rate": 0.089
        }
      ]
    },
    "financial_metrics": {
      "total_transaction_volume": 2345678.90,
      "avg_subscription_price": 49.99,
      "churn_rate": 0.045,
      "creator_earnings": 987654.32,
      "platform_commission": 123456.78
    },
    "time_series": {
      "daily_new_users": [234, 267, 189, 345, 298, 234, 189],
      "daily_revenue": [12345.67, 13456.78, 11234.56, 14567.89],
      "daily_content_creation": [89, 76, 95, 102, 87, 91, 88]
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "267ms",
    "cache_hit": false
  }
}
```

### 2.2 用户行为分析

**接口路径**: `GET /admin/analytics/users`

**描述**: 获取平台用户行为分析数据，包括用户活跃度、留存率和行为模式。

**权限要求**: 需要管理员权限

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| user_type | string | 否 | 用户类型：all, creators, readers，默认all |
| metric | string | 否 | 主要指标：activity, retention, engagement |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "user_activity": {
      "daily_active_users": 34567,
      "weekly_active_users": 89456,
      "monthly_active_users": 125680,
      "dau_wau_ratio": 0.387,
      "dau_mau_ratio": 0.275
    },
    "user_retention": {
      "day_1_retention": 0.78,
      "day_7_retention": 0.45,
      "day_30_retention": 0.23,
      "cohort_analysis": [
        {
          "cohort_month": "2025-08",
          "users_count": 2340,
          "retention_rates": [1.0, 0.78, 0.56, 0.34, 0.23]
        }
      ]
    },
    "user_segments": {
      "by_engagement_level": {
        "high_engagement": {
          "count": 12456,
          "percentage": 0.099,
          "avg_session_duration": 680
        },
        "medium_engagement": {
          "count": 45678,
          "percentage": 0.363,
          "avg_session_duration": 420
        },
        "low_engagement": {
          "count": 67546,
          "percentage": 0.538,
          "avg_session_duration": 180
        }
      },
      "by_subscription_status": {
        "premium_subscribers": {
          "count": 23451,
          "percentage": 0.187,
          "avg_revenue_per_user": 89.45
        },
        "free_users": {
          "count": 102229,
          "percentage": 0.813,
          "conversion_rate": 0.156
        }
      }
    },
    "behavior_patterns": {
      "peak_usage_hours": [9, 12, 14, 20, 22],
      "popular_features": [
        {
          "feature": "文章阅读",
          "usage_rate": 0.89
        },
        {
          "feature": "评论互动",
          "usage_rate": 0.45
        }
      ],
      "user_journey": {
        "avg_time_to_subscription": 7.5,
        "conversion_funnel": {
          "visits": 100000,
          "signups": 15000,
          "first_article_read": 12000,
          "subscriptions": 2340
        }
      }
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "345ms",
    "cache_hit": true
  }
}
```

### 2.3 内容统计分析

**接口路径**: `GET /admin/analytics/content`

**描述**: 获取平台内容统计分析数据，包括内容质量、分类分布和性能指标。

**权限要求**: 需要管理员权限

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| content_type | string | 否 | 内容类型：articles, videos, audio，默认articles |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "content_overview": {
      "total_articles": 45672,
      "published_this_period": 2340,
      "avg_articles_per_day": 78,
      "total_views": 1234567,
      "total_engagements": 567890
    },
    "content_quality": {
      "avg_article_length": 2340,
      "avg_reading_time": 8.5,
      "content_score_distribution": {
        "excellent": 0.23,
        "good": 0.45,
        "average": 0.28,
        "poor": 0.04
      },
      "moderation_statistics": {
        "auto_approved": 0.87,
        "manual_review": 0.11,
        "rejected": 0.02
      }
    },
    "category_performance": [
      {
        "category": "技术教程",
        "article_count": 8945,
        "total_views": 234567,
        "avg_engagement_rate": 0.089,
        "revenue_share": 0.234
      },
      {
        "category": "行业分析",
        "article_count": 6789,
        "total_views": 189456,
        "avg_engagement_rate": 0.076,
        "revenue_share": 0.189
      }
    ],
    "creator_performance": {
      "top_creators": [
        {
          "creator_id": 123,
          "username": "tech_guru",
          "articles_count": 156,
          "total_views": 45672,
          "total_revenue": 12345.67,
          "avg_engagement_rate": 0.094
        }
      ],
      "creator_tiers": {
        "tier_1": {
          "count": 145,
          "min_followers": 10000,
          "avg_revenue": 5678.90
        },
        "tier_2": {
          "count": 567,
          "min_followers": 1000,
          "avg_revenue": 1234.56
        }
      }
    },
    "content_trends": {
      "trending_topics": [
        {
          "topic": "人工智能",
          "mention_count": 567,
          "growth_rate": 0.234
        }
      ],
      "viral_content": [
        {
          "article_id": 789,
          "title": "AI革命的未来展望",
          "viral_score": 0.95,
          "views_growth": 10.5
        }
      ]
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "298ms",
    "cache_hit": false
  }
}
```

### 2.4 平台收入分析

**接口路径**: `GET /admin/analytics/revenue`

**描述**: 获取平台收入分析数据，包括总收入、佣金分成和增长趋势。

**权限要求**: 需要管理员权限

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| time_range | string | 否 | 时间范围：7d, 30d, 90d, 1y，默认30d |
| currency | string | 否 | 货币单位：CNY, USD，默认CNY |
| breakdown | string | 否 | 分解维度：creator, category, plan，默认creator |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "revenue_overview": {
      "total_revenue": 1234567.89,
      "platform_commission": 123456.78,
      "creator_earnings": 1111111.11,
      "commission_rate": 0.10,
      "avg_transaction_value": 49.99,
      "total_transactions": 24702
    },
    "revenue_breakdown": {
      "by_subscription_plan": [
        {
          "plan": "月度基础版",
          "subscribers": 15678,
          "revenue": 470340.00,
          "percentage": 0.381
        },
        {
          "plan": "年度高级版",
          "subscribers": 7890,
          "revenue": 631200.00,
          "percentage": 0.511
        }
      ],
      "by_creator_tier": [
        {
          "tier": "顶级创作者",
          "creator_count": 145,
          "revenue": 567890.12,
          "avg_revenue_per_creator": 3916.48
        }
      ],
      "by_category": [
        {
          "category": "技术教程",
          "revenue": 289456.78,
          "percentage": 0.234,
          "growth_rate": 0.156
        }
      ]
    },
    "payment_analytics": {
      "payment_methods": {
        "微信支付": 0.45,
        "支付宝": 0.38,
        "银行卡": 0.15,
        "其他": 0.02
      },
      "transaction_success_rate": 0.967,
      "avg_processing_time": 2.3,
      "failed_transactions": {
        "count": 234,
        "failure_rate": 0.033,
        "main_reasons": [
          {
            "reason": "余额不足",
            "percentage": 0.45
          },
          {
            "reason": "网络超时",
            "percentage": 0.32
          }
        ]
      }
    },
    "financial_trends": {
      "monthly_revenue": [
        {
          "month": "2025-07",
          "revenue": 998765.43,
          "growth_rate": 0.123
        },
        {
          "month": "2025-08",
          "revenue": 1123456.78,
          "growth_rate": 0.125
        }
      ],
      "revenue_forecast": {
        "next_month_estimate": 1345678.90,
        "confidence_level": 0.85,
        "growth_projection": 0.089
      }
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "234ms",
    "cache_hit": false
  }
}
```

---

## 3. 通知系统接口

### 3.1 通知管理

**接口路径**: `GET /notifications`

**描述**: 获取用户通知列表，支持分页和状态筛选。

#### 请求参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| status | string | 否 | 通知状态：all, unread, read，默认all |
| type | string | 否 | 通知类型：system, content, social, payment |
| page | integer | 否 | 页码，默认1 |
| limit | integer | 否 | 每页数量，默认20，最大100 |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "notification_id": 12345,
        "type": "content",
        "title": "您的文章获得了新的评论",
        "content": "用户 @john_doe 对您的文章《深入理解机器学习》发表了评论",
        "is_read": false,
        "created_at": "2025-09-15T08:30:00Z",
        "action_url": "/articles/789#comment-456",
        "metadata": {
          "article_id": 789,
          "comment_id": 456,
          "user_id": 234
        }
      },
      {
        "notification_id": 12346,
        "type": "social",
        "title": "新的关注者",
        "content": "用户 @jane_smith 开始关注您",
        "is_read": true,
        "created_at": "2025-09-15T07:15:00Z",
        "action_url": "/users/567",
        "metadata": {
          "user_id": 567
        }
      }
    ],
    "summary": {
      "total_notifications": 156,
      "unread_count": 23,
      "today_count": 8
    },
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 156,
      "pages": 8
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z",
    "query_time": "45ms",
    "cache_hit": true
  }
}
```

### 3.2 标记通知已读

**接口路径**: `PUT /notifications/{notification_id}/read`

**描述**: 标记单个通知为已读状态。

#### 路径参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| notification_id | integer | 是 | 通知ID |

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "notification_id": 12345,
    "is_read": true,
    "read_at": "2025-09-15T01:07:11Z"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

### 3.3 全部标记已读

**接口路径**: `PUT /notifications/read-all`

**描述**: 将所有未读通知标记为已读。

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "marked_count": 23,
    "operation_time": "2025-09-15T01:07:11Z"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

### 3.4 推送设置管理

**接口路径**: `GET /notification-settings`

**描述**: 获取用户的通知推送设置。

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "push_settings": {
      "email_notifications": {
        "enabled": true,
        "types": {
          "new_follower": true,
          "article_comment": true,
          "article_like": false,
          "subscription_reminder": true,
          "system_updates": true
        },
        "frequency": "immediate"
      },
      "push_notifications": {
        "enabled": true,
        "types": {
          "new_follower": true,
          "article_comment": true,
          "article_like": false,
          "subscription_reminder": true,
          "system_updates": false
        }
      },
      "in_app_notifications": {
        "enabled": true,
        "types": {
          "new_follower": true,
          "article_comment": true,
          "article_like": true,
          "subscription_reminder": true,
          "system_updates": true
        }
      }
    },
    "quiet_hours": {
      "enabled": true,
      "start_time": "22:00",
      "end_time": "08:00",
      "timezone": "Asia/Shanghai"
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

### 3.5 更新推送设置

**接口路径**: `PUT /notification-settings`

**描述**: 更新用户的通知推送设置。

#### 请求参数

```json
{
  "push_settings": {
    "email_notifications": {
      "enabled": true,
      "types": {
        "new_follower": true,
        "article_comment": true,
        "article_like": false,
        "subscription_reminder": true,
        "system_updates": true
      },
      "frequency": "daily_digest"
    },
    "push_notifications": {
      "enabled": true,
      "types": {
        "new_follower": true,
        "article_comment": true,
        "article_like": false,
        "subscription_reminder": true,
        "system_updates": false
      }
    }
  },
  "quiet_hours": {
    "enabled": true,
    "start_time": "22:00",
    "end_time": "08:00"
  }
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "settings_updated": true,
    "updated_at": "2025-09-15T01:07:11Z"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

### 3.6 设备Token管理

**接口路径**: `POST /push-tokens`

**描述**: 注册设备推送Token，用于移动端推送。

#### 请求参数

```json
{
  "token": "device_push_token_string",
  "platform": "ios",  // ios, android, web
  "device_info": {
    "device_id": "unique_device_identifier",
    "app_version": "1.0.0",
    "os_version": "iOS 17.0"
  }
}
```

#### 响应示例

**成功响应 (201)**:
```json
{
  "success": true,
  "data": {
    "token_id": 789,
    "token": "device_push_token_string",
    "platform": "ios",
    "registered_at": "2025-09-15T01:07:11Z",
    "is_active": true
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 4. 数据导出和报表接口

### 4.1 数据导出

**接口路径**: `POST /analytics/export`

**描述**: 导出分析数据，支持多种格式和自定义时间范围。

#### 请求参数

```json
{
  "data_type": "analytics_overview",  // analytics_overview, articles, revenue, audience
  "time_range": "30d",
  "format": "csv",  // csv, json, excel
  "filters": {
    "category": "技术教程",
    "minimum_views": 100
  },
  "email_delivery": true,
  "email": "user@example.com"
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "export_id": "exp_12345",
    "status": "processing",
    "estimated_completion": "2025-09-15T01:10:00Z",
    "download_url": null,
    "file_size": null,
    "expires_at": "2025-09-22T01:07:11Z"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

### 4.2 报表生成

**接口路径**: `POST /analytics/reports`

**描述**: 生成定制化数据报表，支持多种可视化图表。

#### 请求参数

```json
{
  "report_type": "monthly_summary",  // monthly_summary, quarterly_review, annual_report
  "time_range": "30d",
  "sections": [
    "overview",
    "content_performance",
    "audience_insights",
    "revenue_analysis"
  ],
  "visualization": {
    "include_charts": true,
    "chart_types": ["line", "bar", "pie"],
    "format": "pdf"
  },
  "email_delivery": true
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "report_id": "rpt_67890",
    "status": "generating",
    "estimated_completion": "2025-09-15T01:15:00Z",
    "download_url": null,
    "expires_at": "2025-09-22T01:07:11Z"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 5. 实时数据和Webhook

### 5.1 实时数据流

**接口路径**: `WebSocket /analytics/realtime`

**描述**: 建立WebSocket连接，接收实时数据更新。

#### 连接参数

| 参数名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| metrics | string | 否 | 订阅的指标类型，多个用逗号分隔 |
| interval | integer | 否 | 更新间隔(秒)，默认5秒 |

#### 实时数据格式

```json
{
  "type": "realtime_update",
  "timestamp": "2025-09-15T01:07:11Z",
  "data": {
    "current_online_users": 1234,
    "articles_views_last_hour": 567,
    "new_subscriptions_today": 23,
    "revenue_today": 1234.56
  }
}
```

### 5.2 数据Webhook

**接口路径**: `POST /analytics/webhooks`

**描述**: 配置数据变化的Webhook通知。

#### 请求参数

```json
{
  "webhook_url": "https://your-app.com/webhooks/analytics",
  "events": [
    "milestone_reached",
    "threshold_exceeded",
    "anomaly_detected"
  ],
  "filters": {
    "metric": "revenue",
    "threshold": 10000
  },
  "secret": "webhook_secret_key"
}
```

---

## 6. 数据隐私和安全

### 6.1 数据隐私保护

#### GDPR合规
- **数据最小化**: 仅收集业务必需的用户数据
- **用户同意**: 明确的数据使用同意机制
- **数据删除权**: 支持用户删除个人数据请求
- **数据可携带权**: 提供数据导出功能

#### 数据匿名化
- **个人信息脱敏**: 统计数据中移除个人身份信息
- **聚合数据**: 仅提供聚合后的统计数据
- **差分隐私**: 在统计查询中添加噪声保护

### 6.2 访问控制

#### 角色权限
```json
{
  "roles": {
    "creator": {
      "permissions": [
        "analytics:own_data",
        "notifications:manage"
      ]
    },
    "admin": {
      "permissions": [
        "analytics:platform_data",
        "admin:user_analytics",
        "admin:content_analytics",
        "admin:revenue_analytics"
      ]
    }
  }
}
```

#### API限流规则
- **创作者用户**: 每分钟100次请求
- **管理员用户**: 每分钟500次请求
- **数据导出**: 每小时5次请求
- **实时数据**: 每用户最多3个并发连接

---

## 7. 性能优化和缓存策略

### 7.1 缓存机制

#### 多层缓存架构
- **Redis缓存**: 热点数据3600秒缓存
- **CDN缓存**: 静态报表文件24小时缓存
- **应用层缓存**: 计算结果300秒内存缓存

#### 缓存失效策略
```json
{
  "cache_invalidation": {
    "real_time_data": "5s",
    "hourly_aggregation": "1h",
    "daily_aggregation": "24h",
    "monthly_reports": "30d"
  }
}
```

### 7.2 查询优化

#### 数据库索引
- **时间序列索引**: created_at, updated_at字段复合索引
- **用户维度索引**: user_id, creator_id相关查询优化
- **聚合查询索引**: 支持GROUP BY操作的覆盖索引

#### 分页优化
- **游标分页**: 大数据量查询使用游标分页
- **预聚合**: 常用统计数据预先计算存储
- **异步处理**: 复杂报表生成使用异步队列

---

## 8. 监控和告警

### 8.1 系统监控指标

#### API性能监控
- **响应时间**: P50, P95, P99响应时间监控
- **错误率**: API错误率和错误类型统计
- **吞吐量**: QPS和并发数监控
- **可用性**: 服务可用性SLA监控

#### 数据质量监控
- **数据一致性**: 跨表数据一致性检查
- **数据完整性**: 必要字段缺失监控
- **异常检测**: 数据异常波动告警

### 8.2 告警配置

#### 关键指标告警
```json
{
  "alerts": {
    "high_error_rate": {
      "threshold": "error_rate > 0.05",
      "duration": "5m",
      "notification": ["email", "slack"]
    },
    "slow_response": {
      "threshold": "p95_response_time > 1000ms",
      "duration": "3m",
      "notification": ["email"]
    },
    "data_anomaly": {
      "threshold": "data_change > 50%",
      "duration": "1m",
      "notification": ["email", "sms"]
    }
  }
}
```

---

## 9. 错误处理和故障恢复

### 9.1 错误分类和处理

#### 系统级错误
- **数据库连接失败**: 自动重试机制，最大重试3次
- **缓存服务不可用**: 降级到数据库查询
- **第三方服务超时**: 断路器模式，快速失败

#### 业务级错误
- **权限不足**: 返回403状态码和详细错误信息
- **数据不存在**: 返回404状态码和建议操作
- **参数验证失败**: 返回400状态码和具体验证错误

### 9.2 故障恢复策略

#### 服务降级
```json
{
  "degradation_rules": {
    "high_load": {
      "disable_real_time_updates": true,
      "reduce_data_granularity": true,
      "enable_aggressive_caching": true
    },
    "partial_failure": {
      "fallback_to_cached_data": true,
      "skip_non_essential_metrics": true
    }
  }
}
```

#### 数据备份和恢复
- **实时备份**: 关键数据实时同步备份
- **定期快照**: 每日数据快照备份
- **灾难恢复**: RTO < 1小时，RPO < 15分钟

---

## 附录

### A. API版本兼容性

#### 版本策略
- **向后兼容**: 保证API向后兼容性至少1年
- **废弃通知**: 废弃API提前3个月通知
- **平滑升级**: 支持API版本并行运行

### B. 开发工具和SDK

#### 官方SDK支持
- **JavaScript SDK**: 完整的前端数据分析SDK
- **Python SDK**: 服务端数据处理SDK
- **React Hooks**: 专用React组件库

#### 第三方集成
- **Tableau连接器**: 直接连接Tableau进行数据分析
- **Power BI连接器**: 支持Microsoft Power BI集成
- **Google Analytics**: 数据对比和补充分析

### C. 数据字典

#### 核心指标定义
- **活跃用户**: 30天内有任何操作的用户
- **参与度**: (点赞+评论+分享+收藏) / 总浏览量
- **转化率**: 订阅用户数 / 总访问用户数
- **留存率**: 特定时间段后仍活跃的用户比例
- **LTV**: 用户生命周期价值
- **CAC**: 客户获取成本

### D. 更新日志

#### v1.0.0 (2025-09-15)
- ✅ 完成创作者分析API (5个接口)
- ✅ 完成平台监控API (4个接口)
- ✅ 完成通知系统API (6个接口)
- ✅ 实现数据导出和报表功能
- ✅ 建立实时数据推送机制
- ✅ 完善数据隐私保护机制
- 🔒 实施完整的访问控制策略
- ⚡ 优化查询性能和缓存机制

---

**文档版本**: v1.0.0
**维护团队**: ShareStack 数据团队
**最后更新**: 2025-09-15
**联系邮箱**: data-api@sharestack.com

*本文档基于 OpenAPI 3.0 规范编写，提供完整的数据分析API接口规范，支持创作者洞察、平台监控和智能通知功能。*