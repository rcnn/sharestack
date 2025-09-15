# ShareStack 社交互动API文档

**版本**: v1.0
**创建时间**: 2025-09-15
**最后更新**: 2025-09-15

## 概述

ShareStack社交互动模块提供完整的社交功能API，包括用户关系管理、内容互动、评论系统和内容发现推荐。本文档详细描述了24个核心API接口，支持知识付费平台的社区建设和用户互动体验。

## 目录

- [1. 社交关系接口](#1-社交关系接口)
- [2. 互动功能接口](#2-互动功能接口)
- [3. 评论系统接口](#3-评论系统接口)
- [4. 内容发现接口](#4-内容发现接口)
- [5. 推荐算法规范](#5-推荐算法规范)
- [6. 用户行为追踪](#6-用户行为追踪)
- [7. 社区管理机制](#7-社区管理机制)
- [8. 数据模型](#8-数据模型)
- [9. 错误处理](#9-错误处理)

---

## 1. 社交关系接口

### 1.1 关注用户

**接口描述**: 创建用户关注关系

```http
POST /api/v1/follows
```

**请求参数**:
```json
{
  "target_user_id": 123,
  "notification_enabled": true
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": 456,
    "follower_id": 789,
    "following_id": 123,
    "notification_enabled": true,
    "created_at": "2025-09-15T10:00:00Z"
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T10:00:00Z"
  }
}
```

**业务规则**:
- 用户不能关注自己
- 重复关注返回已存在的关注记录
- 支持通知开关设置
- 关注后触发推荐算法更新

### 1.2 取消关注

**接口描述**: 删除用户关注关系

```http
DELETE /api/v1/follows/{user_id}
```

**路径参数**:
- `user_id`: 被取消关注的用户ID

**响应示例**:
```json
{
  "success": true,
  "data": {
    "message": "取消关注成功"
  }
}
```

### 1.3 获取关注者列表 (粉丝)

**接口描述**: 获取指定用户的粉丝列表

```http
GET /api/v1/users/{id}/followers
```

**查询参数**:
- `page`: 页码 (默认1)
- `limit`: 每页数量 (默认20，最大100)
- `sort`: 排序方式 (follow_time_desc, follow_time_asc)

**响应示例**:
```json
{
  "success": true,
  "data": {
    "followers": [
      {
        "id": 789,
        "username": "john_doe",
        "display_name": "John Doe",
        "avatar_url": "https://cdn.sharestack.com/avatars/789.jpg",
        "follower_count": 150,
        "following_count": 300,
        "is_verified": false,
        "followed_at": "2025-09-15T09:00:00Z",
        "mutual_follow": true
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 145,
      "pages": 8
    }
  }
}
```

### 1.4 获取关注列表

**接口描述**: 获取指定用户关注的人员列表

```http
GET /api/v1/users/{id}/following
```

**查询参数**: 同关注者列表

**响应示例**: 类似关注者列表结构

---

## 2. 互动功能接口

### 2.1 点赞内容

**接口描述**: 对文章进行点赞操作

```http
POST /api/v1/articles/{id}/like
```

**路径参数**:
- `id`: 文章ID

**响应示例**:
```json
{
  "success": true,
  "data": {
    "like_id": 123,
    "article_id": 456,
    "user_id": 789,
    "created_at": "2025-09-15T10:00:00Z",
    "total_likes": 128
  }
}
```

**业务规则**:
- 重复点赞返回已存在记录
- 点赞后增加文章热度分数
- 触发用户行为事件记录
- 通知文章作者 (根据设置)

### 2.2 取消点赞

**接口描述**: 取消对文章的点赞

```http
DELETE /api/v1/articles/{id}/like
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "message": "取消点赞成功",
    "total_likes": 127
  }
}
```

### 2.3 收藏内容

**接口描述**: 将文章添加到个人收藏

```http
POST /api/v1/articles/{id}/bookmark
```

**请求参数**:
```json
{
  "collection_id": 123,
  "notes": "个人学习笔记",
  "tags": ["学习", "技术"]
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "bookmark_id": 456,
    "article_id": 789,
    "user_id": 123,
    "collection_id": 123,
    "notes": "个人学习笔记",
    "tags": ["学习", "技术"],
    "created_at": "2025-09-15T10:00:00Z"
  }
}
```

### 2.4 取消收藏

**接口描述**: 从收藏中移除文章

```http
DELETE /api/v1/articles/{id}/bookmark
```

### 2.5 获取个人收藏列表

**接口描述**: 获取用户的收藏文章列表

```http
GET /api/v1/users/bookmarks
```

**查询参数**:
- `collection_id`: 收藏夹ID (可选)
- `tags`: 标签过滤 (可选)
- `sort`: 排序方式 (bookmark_time_desc, bookmark_time_asc, article_popularity)
- `page`: 页码
- `limit`: 每页数量

**响应示例**:
```json
{
  "success": true,
  "data": {
    "bookmarks": [
      {
        "bookmark_id": 456,
        "article": {
          "id": 789,
          "title": "深度学习入门指南",
          "excerpt": "本文介绍深度学习的基础概念...",
          "author": {
            "id": 123,
            "username": "ai_expert",
            "display_name": "AI专家"
          },
          "cover_image": "https://cdn.sharestack.com/covers/789.jpg",
          "reading_time": 15,
          "like_count": 128,
          "bookmark_count": 45
        },
        "notes": "个人学习笔记",
        "tags": ["学习", "技术"],
        "bookmarked_at": "2025-09-15T10:00:00Z"
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "pages": 3
    }
  }
}
```

---

## 3. 评论系统接口

### 3.1 创建评论

**接口描述**: 对文章发表评论或回复

```http
POST /api/v1/articles/{id}/comments
```

**请求参数**:
```json
{
  "content": "这篇文章写得很好，学到了很多！",
  "parent_id": null,
  "mentions": [123, 456],
  "reply_to_user_id": null
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": 789,
    "article_id": 123,
    "user_id": 456,
    "content": "这篇文章写得很好，学到了很多！",
    "parent_id": null,
    "reply_to_user_id": null,
    "mentions": [123, 456],
    "like_count": 0,
    "reply_count": 0,
    "status": "published",
    "created_at": "2025-09-15T10:00:00Z",
    "updated_at": "2025-09-15T10:00:00Z",
    "user": {
      "id": 456,
      "username": "reader123",
      "display_name": "读者123",
      "avatar_url": "https://cdn.sharestack.com/avatars/456.jpg"
    }
  }
}
```

### 3.2 获取评论列表

**接口描述**: 获取文章的评论列表

```http
GET /api/v1/articles/{id}/comments
```

**查询参数**:
- `sort`: 排序方式 (newest, oldest, most_liked, most_replied)
- `page`: 页码
- `limit`: 每页数量
- `include_replies`: 是否包含回复 (true/false)

**响应示例**:
```json
{
  "success": true,
  "data": {
    "comments": [
      {
        "id": 789,
        "content": "这篇文章写得很好！",
        "user": {
          "id": 456,
          "username": "reader123",
          "display_name": "读者123",
          "avatar_url": "https://cdn.sharestack.com/avatars/456.jpg",
          "is_verified": false
        },
        "like_count": 5,
        "reply_count": 2,
        "is_liked": false,
        "created_at": "2025-09-15T10:00:00Z",
        "replies": [
          {
            "id": 790,
            "content": "同意你的观点！",
            "user": {
              "id": 789,
              "username": "commenter",
              "display_name": "评论者"
            },
            "like_count": 1,
            "created_at": "2025-09-15T10:05:00Z"
          }
        ]
      }
    ]
  },
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "pages": 3
    }
  }
}
```

### 3.3 更新评论

**接口描述**: 编辑已发表的评论

```http
PUT /api/v1/comments/{id}
```

**请求参数**:
```json
{
  "content": "更新后的评论内容"
}
```

### 3.4 删除评论

**接口描述**: 删除评论 (仅作者或管理员)

```http
DELETE /api/v1/comments/{id}
```

### 3.5 点赞评论

**接口描述**: 对评论进行点赞

```http
POST /api/v1/comments/{id}/like
```

### 3.6 举报评论

**接口描述**: 举报不当评论

```http
POST /api/v1/comments/{id}/report
```

**请求参数**:
```json
{
  "reason": "spam",
  "description": "这是垃圾评论",
  "category": "inappropriate_content"
}
```

### 3.7 @提及通知

**接口描述**: 处理评论中的@提及功能

提及用户时自动发送通知，支持用户名自动补全。

---

## 4. 内容发现接口

### 4.1 全文搜索

**接口描述**: 搜索文章、用户和标签

```http
GET /api/v1/search
```

**查询参数**:
- `q`: 搜索关键词 (必需)
- `type`: 搜索类型 (articles, users, tags, all)
- `category`: 分类筛选
- `author_id`: 作者筛选
- `date_range`: 日期范围 (week, month, year, custom)
- `sort`: 排序方式 (relevance, date, popularity)
- `page`: 页码
- `limit`: 每页数量

**响应示例**:
```json
{
  "success": true,
  "data": {
    "results": {
      "articles": [
        {
          "id": 123,
          "title": "机器学习入门指南",
          "excerpt": "本文介绍机器学习的基础概念...",
          "author": {
            "id": 456,
            "username": "ml_expert",
            "display_name": "ML专家"
          },
          "category": "技术",
          "tags": ["机器学习", "AI", "教程"],
          "reading_time": 15,
          "like_count": 128,
          "bookmark_count": 45,
          "created_at": "2025-09-15T10:00:00Z",
          "relevance_score": 0.95
        }
      ],
      "users": [
        {
          "id": 456,
          "username": "ml_expert",
          "display_name": "ML专家",
          "bio": "机器学习工程师，专注于深度学习研究",
          "follower_count": 1520,
          "article_count": 45,
          "is_verified": true
        }
      ],
      "tags": [
        {
          "name": "机器学习",
          "article_count": 234,
          "follower_count": 1520
        }
      ]
    },
    "suggestions": ["机器学习算法", "深度学习", "神经网络"],
    "total_results": {
      "articles": 45,
      "users": 8,
      "tags": 12
    }
  }
}
```

### 4.2 搜索建议

**接口描述**: 获取搜索自动补全建议

```http
GET /api/v1/search/suggestions
```

**查询参数**:
- `q`: 输入的关键词
- `type`: 建议类型 (keywords, users, tags)
- `limit`: 建议数量 (默认10)

### 4.3 热门搜索

**接口描述**: 获取热门搜索关键词

```http
GET /api/v1/search/trending
```

### 4.4 个性化推荐

**接口描述**: 基于用户行为的个性化内容推荐

```http
GET /api/v1/recommendations/feed
```

**查询参数**:
- `algorithm`: 推荐算法 (collaborative, content_based, hybrid)
- `diversity`: 多样性系数 (0.0-1.0)
- `page`: 页码
- `limit`: 每页数量

**响应示例**:
```json
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "article": {
          "id": 123,
          "title": "推荐的文章标题",
          "author": {...},
          "category": "技术",
          "reading_time": 10
        },
        "recommendation_score": 0.92,
        "reason": "基于你对机器学习的兴趣",
        "algorithm": "collaborative_filtering"
      }
    ],
    "algorithm_info": {
      "primary_algorithm": "hybrid",
      "factors": ["reading_history", "follow_relationships", "content_similarity"],
      "diversity_score": 0.7
    }
  }
}
```

### 4.5 热门内容

**接口描述**: 获取平台热门内容

```http
GET /api/v1/recommendations/trending
```

**查询参数**:
- `time_range`: 时间范围 (day, week, month)
- `category`: 分类筛选
- `metric`: 热门指标 (views, likes, comments, bookmarks, overall)

### 4.6 推荐创作者

**接口描述**: 推荐值得关注的创作者

```http
GET /api/v1/recommendations/creators
```

### 4.7 相似内容

**接口描述**: 基于当前文章推荐相似内容

```http
GET /api/v1/recommendations/similar
```

**查询参数**:
- `article_id`: 当前文章ID
- `similarity_threshold`: 相似度阈值 (0.0-1.0)
- `limit`: 推荐数量

### 4.8 分类浏览

**接口描述**: 按分类浏览内容

```http
GET /api/v1/categories/{id}/articles
```

**查询参数**:
- `sort`: 排序方式 (newest, popular, trending)
- `featured`: 是否只显示精选内容
- `page`: 页码
- `limit`: 每页数量

---

## 5. 推荐算法规范

### 5.1 算法输入数据格式

**用户特征数据**:
```json
{
  "user_id": 123,
  "features": {
    "demographics": {
      "age_range": "25-34",
      "profession": "软件工程师",
      "interests": ["技术", "创业", "投资"]
    },
    "behavior": {
      "reading_time_avg": 12.5,
      "preferred_content_length": "medium",
      "active_time_slots": ["morning", "evening"],
      "device_preference": "mobile"
    },
    "engagement": {
      "like_rate": 0.15,
      "comment_rate": 0.05,
      "share_rate": 0.02,
      "bookmark_rate": 0.08
    }
  }
}
```

**内容特征数据**:
```json
{
  "article_id": 456,
  "features": {
    "content": {
      "category": "技术",
      "tags": ["机器学习", "Python"],
      "reading_time": 15,
      "difficulty_level": "intermediate",
      "content_type": "tutorial"
    },
    "engagement": {
      "view_count": 1520,
      "like_count": 128,
      "comment_count": 45,
      "bookmark_count": 67,
      "engagement_rate": 0.12
    },
    "quality": {
      "content_score": 0.85,
      "author_reputation": 0.92,
      "freshness_score": 0.78
    }
  }
}
```

### 5.2 算法输出格式

**推荐结果**:
```json
{
  "user_id": 123,
  "recommendations": [
    {
      "article_id": 456,
      "score": 0.92,
      "confidence": 0.87,
      "reasons": [
        {
          "type": "content_similarity",
          "weight": 0.4,
          "description": "与你最近阅读的技术文章相似"
        },
        {
          "type": "collaborative_filtering",
          "weight": 0.3,
          "description": "相似用户也喜欢这篇文章"
        },
        {
          "type": "author_preference",
          "weight": 0.3,
          "description": "来自你关注的作者"
        }
      ]
    }
  ],
  "algorithm_metadata": {
    "model_version": "v2.1",
    "inference_time_ms": 45,
    "diversity_score": 0.7,
    "novelty_score": 0.6
  }
}
```

### 5.3 冷启动策略

**新用户推荐**:
1. 基于注册信息的初始推荐
2. 热门内容推荐
3. 分类内容采样
4. 快速用户画像建立

**新内容推荐**:
1. 基于内容特征的相似度推荐
2. 作者历史表现权重
3. 内容质量评分
4. 时效性加权

---

## 6. 用户行为追踪

### 6.1 行为事件定义

**阅读行为事件**:
```json
{
  "event_type": "article_view",
  "user_id": 123,
  "article_id": 456,
  "timestamp": "2025-09-15T10:00:00Z",
  "session_id": "sess_789",
  "properties": {
    "source": "recommendation",
    "referrer": "home_feed",
    "reading_time": 180,
    "scroll_depth": 0.85,
    "completion_rate": 0.9,
    "device": "mobile",
    "location": "CN"
  }
}
```

**互动行为事件**:
```json
{
  "event_type": "article_like",
  "user_id": 123,
  "article_id": 456,
  "timestamp": "2025-09-15T10:05:00Z",
  "properties": {
    "reading_time_before_like": 120,
    "scroll_position": 0.6,
    "is_first_like_on_article": true
  }
}
```

### 6.2 事件上报接口

**批量事件上报**:
```http
POST /api/v1/analytics/events/batch
```

**请求参数**:
```json
{
  "events": [
    {
      "event_type": "article_view",
      "user_id": 123,
      "properties": {...}
    }
  ]
}
```

### 6.3 实时行为分析

**用户行为模式识别**:
- 阅读偏好分析
- 活跃时间段统计
- 内容消费路径追踪
- 转化漏斗分析

---

## 7. 社区管理机制

### 7.1 内容审核规则

**自动审核机制**:
1. **关键词过滤**: 敏感词汇检测
2. **语义分析**: AI内容理解与分类
3. **重复内容检测**: 防止垃圾内容
4. **质量评分**: 内容价值评估

**审核状态流转**:
```
提交 → 自动审核 → [通过/待人工审核/拒绝]
          ↓
       人工审核 → [通过/拒绝/需修改]
```

### 7.2 用户举报处理

**举报分类**:
- `spam`: 垃圾内容
- `harassment`: 骚扰他人
- `inappropriate_content`: 不当内容
- `copyright_violation`: 版权侵权
- `misinformation`: 虚假信息

**处理流程**:
```http
POST /api/v1/reports
```

**请求参数**:
```json
{
  "target_type": "comment",
  "target_id": 123,
  "reason": "spam",
  "description": "详细描述举报原因",
  "evidence_urls": ["https://example.com/screenshot.jpg"]
}
```

### 7.3 用户信誉系统

**信誉分数计算**:
```json
{
  "user_id": 123,
  "reputation_score": 85.5,
  "factors": {
    "content_quality": 0.4,
    "community_engagement": 0.3,
    "violation_history": -0.1,
    "peer_recognition": 0.3,
    "platform_contribution": 0.1
  },
  "level": "trusted_contributor",
  "privileges": [
    "skip_content_review",
    "moderate_comments",
    "access_beta_features"
  ]
}
```

### 7.4 反垃圾机制

**检测指标**:
1. **频率限制**: 操作频率异常检测
2. **内容相似度**: 重复内容识别
3. **行为模式**: 机器人行为检测
4. **IP声誉**: 来源IP信誉评估

**防护措施**:
- 验证码验证
- 临时限制操作
- 账号风险评级
- 内容延迟发布

---

## 8. 数据模型

### 8.1 关注关系表

```sql
CREATE TABLE follows (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  follower_id BIGINT NOT NULL,
  following_id BIGINT NOT NULL,
  notification_enabled BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_follower (follower_id),
  INDEX idx_following (following_id),
  UNIQUE KEY uk_follow_relation (follower_id, following_id)
);
```

### 8.2 点赞表

```sql
CREATE TABLE likes (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  target_type ENUM('article', 'comment') NOT NULL,
  target_id BIGINT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user_likes (user_id),
  INDEX idx_target (target_type, target_id),
  UNIQUE KEY uk_user_like (user_id, target_type, target_id)
);
```

### 8.3 收藏表

```sql
CREATE TABLE bookmarks (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  article_id BIGINT NOT NULL,
  collection_id BIGINT DEFAULT NULL,
  notes TEXT,
  tags JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user_bookmarks (user_id),
  INDEX idx_article_bookmarks (article_id),
  UNIQUE KEY uk_user_article_bookmark (user_id, article_id)
);
```

### 8.4 评论表

```sql
CREATE TABLE comments (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  article_id BIGINT NOT NULL,
  user_id BIGINT NOT NULL,
  parent_id BIGINT DEFAULT NULL,
  reply_to_user_id BIGINT DEFAULT NULL,
  content TEXT NOT NULL,
  mentions JSON,
  like_count INT DEFAULT 0,
  reply_count INT DEFAULT 0,
  status ENUM('published', 'pending', 'rejected') DEFAULT 'published',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_article_comments (article_id),
  INDEX idx_user_comments (user_id),
  INDEX idx_parent_comments (parent_id)
);
```

### 8.5 用户行为事件表

```sql
CREATE TABLE user_behavior_events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  event_type VARCHAR(50) NOT NULL,
  target_type VARCHAR(20),
  target_id BIGINT,
  session_id VARCHAR(100),
  properties JSON,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_user_events (user_id, timestamp),
  INDEX idx_event_type (event_type, timestamp),
  INDEX idx_target (target_type, target_id)
);
```

---

## 9. 错误处理

### 9.1 错误代码定义

| 错误代码 | HTTP状态码 | 描述 |
|---------|-----------|------|
| FOLLOW_SELF_ERROR | 400 | 不能关注自己 |
| ALREADY_FOLLOWED | 409 | 已经关注该用户 |
| USER_NOT_FOUND | 404 | 用户不存在 |
| ARTICLE_NOT_FOUND | 404 | 文章不存在 |
| COMMENT_NOT_FOUND | 404 | 评论不存在 |
| PERMISSION_DENIED | 403 | 权限不足 |
| RATE_LIMIT_EXCEEDED | 429 | 操作过于频繁 |
| CONTENT_MODERATION_FAILED | 400 | 内容审核不通过 |
| SPAM_DETECTED | 400 | 检测到垃圾内容 |

### 9.2 错误响应格式

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "FOLLOW_SELF_ERROR",
    "message": "不能关注自己",
    "details": {
      "field": "target_user_id",
      "provided_value": 123,
      "current_user_id": 123
    }
  },
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T10:00:00Z",
    "request_id": "req_123456"
  }
}
```

### 9.3 业务异常处理

**关注限制**:
- 每日关注数量限制 (防止批量关注)
- 关注总数上限 (防止异常账号)

**内容互动限制**:
- 点赞频率限制 (防止刷赞)
- 评论频率限制 (防止垃圾评论)
- 内容长度限制 (评论1000字符)

**搜索保护**:
- 查询长度限制 (最多100字符)
- 搜索频率限制 (防止滥用)
- 恶意查询检测

---

## 接口汇总

### 社交关系接口 (4个)
1. `POST /api/v1/follows` - 关注用户
2. `DELETE /api/v1/follows/{user_id}` - 取消关注
3. `GET /api/v1/users/{id}/followers` - 获取关注者列表
4. `GET /api/v1/users/{id}/following` - 获取关注列表

### 互动功能接口 (5个)
1. `POST /api/v1/articles/{id}/like` - 点赞内容
2. `DELETE /api/v1/articles/{id}/like` - 取消点赞
3. `POST /api/v1/articles/{id}/bookmark` - 收藏内容
4. `DELETE /api/v1/articles/{id}/bookmark` - 取消收藏
5. `GET /api/v1/users/bookmarks` - 获取个人收藏列表

### 评论系统接口 (7个)
1. `POST /api/v1/articles/{id}/comments` - 创建评论
2. `GET /api/v1/articles/{id}/comments` - 获取评论列表
3. `PUT /api/v1/comments/{id}` - 更新评论
4. `DELETE /api/v1/comments/{id}` - 删除评论
5. `POST /api/v1/comments/{id}/like` - 点赞评论
6. `POST /api/v1/comments/{id}/report` - 举报评论
7. `POST /api/v1/reports` - 用户举报处理

### 内容发现接口 (8个)
1. `GET /api/v1/search` - 全文搜索
2. `GET /api/v1/search/suggestions` - 搜索建议
3. `GET /api/v1/search/trending` - 热门搜索
4. `GET /api/v1/recommendations/feed` - 个性化推荐
5. `GET /api/v1/recommendations/trending` - 热门内容
6. `GET /api/v1/recommendations/creators` - 推荐创作者
7. `GET /api/v1/recommendations/similar` - 相似内容
8. `GET /api/v1/categories/{id}/articles` - 分类浏览

**总计**: 24个核心社交互动API接口

---

**文档版本**: v1.0
**最后更新**: 2025-09-15
**维护团队**: ShareStack后端开发团队