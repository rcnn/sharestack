# ShareStack 内容管理 API 文档

## 概述

ShareStack 内容管理模块提供完整的文章创作、发布、媒体上传、分类标签管理功能。本文档遵循 OpenAPI 3.0 规范，为所有内容相关接口提供详细的规范说明。

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
| PAYLOAD_TOO_LARGE | 413 | 文件过大 |
| UNSUPPORTED_MEDIA_TYPE | 415 | 不支持的文件类型 |
| RATE_LIMIT_EXCEEDED | 429 | 请求频率超限 |
| INTERNAL_ERROR | 500 | 服务器内部错误 |

---

## 1. 文章管理接口

### 1.1 创建文章

**接口路径**: `POST /articles`

**描述**: 创建新文章，支持草稿模式和立即发布。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "title": "string",                  // 文章标题，必填，1-200字符
  "content": "string",                // 文章内容，必填，支持Markdown/HTML
  "excerpt": "string",                // 文章摘要，可选，0-500字符
  "cover_image": "string",            // 封面图片URL，可选
  "tags": ["string"],                 // 标签数组，可选，最多10个标签
  "category_id": "integer",           // 分类ID，可选
  "visibility": "string",             // 可见性：public|subscribers|paid
  "allow_comments": "boolean",        // 是否允许评论，默认true
  "scheduled_at": "datetime",         // 定时发布时间，可选
  "seo_title": "string",              // SEO标题，可选，0-60字符
  "seo_description": "string",        // SEO描述，可选，0-160字符
  "custom_url": "string",             // 自定义URL别名，可选
  "status": "string"                  // 文章状态：draft|published
}
```

#### 响应示例

**成功响应 (201)**:
```json
{
  "success": true,
  "data": {
    "article_id": 12345,
    "title": "如何构建高性能的React应用",
    "content": "在现代Web开发中...",
    "excerpt": "本文介绍了构建高性能React应用的最佳实践",
    "cover_image": "https://cdn.sharestack.com/covers/12345.jpg",
    "tags": ["React", "性能优化", "前端"],
    "category": {
      "id": 1,
      "name": "前端开发",
      "slug": "frontend"
    },
    "visibility": "public",
    "allow_comments": true,
    "status": "published",
    "seo_title": "高性能React应用构建指南",
    "seo_description": "学习如何构建高性能的React应用",
    "custom_url": "high-performance-react-app",
    "author": {
      "user_id": 67890,
      "username": "react_master",
      "display_name": "React大师",
      "avatar": "https://cdn.sharestack.com/avatars/67890.jpg"
    },
    "stats": {
      "views_count": 0,
      "likes_count": 0,
      "comments_count": 0,
      "bookmarks_count": 0
    },
    "published_at": "2025-09-15T01:07:11Z",
    "created_at": "2025-09-15T01:07:11Z",
    "updated_at": "2025-09-15T01:07:11Z"
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
curl -X POST "https://api.sharestack.com/api/v1/articles" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "title": "如何构建高性能的React应用",
    "content": "在现代Web开发中，React已经成为最受欢迎的前端框架之一...",
    "excerpt": "本文介绍了构建高性能React应用的最佳实践",
    "tags": ["React", "性能优化", "前端"],
    "category_id": 1,
    "visibility": "public",
    "status": "published"
  }'
```

#### JavaScript 示例

```javascript
const createArticle = async (articleData) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch('/api/v1/articles', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(articleData)
  });

  const result = await response.json();
  return result;
};

// 使用示例
const articleData = {
  title: '如何构建高性能的React应用',
  content: '在现代Web开发中，React已经成为最受欢迎的前端框架之一...',
  excerpt: '本文介绍了构建高性能React应用的最佳实践',
  tags: ['React', '性能优化', '前端'],
  category_id: 1,
  visibility: 'public',
  status: 'published'
};

const result = await createArticle(articleData);
if (result.success) {
  console.log('文章创建成功:', result.data.article_id);
}
```

---

### 1.2 获取文章列表

**接口路径**: `GET /articles`

**描述**: 获取文章列表，支持筛选、排序、分页。

#### 查询参数

```
page: 页码，默认1
limit: 每页数量，默认20，最大100
status: 文章状态筛选 (draft|published|archived)
category: 分类筛选
tags: 标签筛选，逗号分隔
author: 作者筛选
visibility: 可见性筛选 (public|subscribers|paid)
sort: 排序方式 (created_at|updated_at|published_at|views|likes)
order: 排序方向 (asc|desc)，默认desc
search: 关键词搜索
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "articles": [
      {
        "article_id": 12345,
        "title": "如何构建高性能的React应用",
        "excerpt": "本文介绍了构建高性能React应用的最佳实践",
        "cover_image": "https://cdn.sharestack.com/covers/12345.jpg",
        "tags": ["React", "性能优化", "前端"],
        "category": {
          "id": 1,
          "name": "前端开发",
          "slug": "frontend"
        },
        "visibility": "public",
        "status": "published",
        "author": {
          "user_id": 67890,
          "username": "react_master",
          "display_name": "React大师",
          "avatar": "https://cdn.sharestack.com/avatars/67890.jpg"
        },
        "stats": {
          "views_count": 1250,
          "likes_count": 89,
          "comments_count": 23,
          "bookmarks_count": 156
        },
        "published_at": "2025-09-15T01:07:11Z",
        "updated_at": "2025-09-15T01:07:11Z"
      }
    ]
  },
  "error": null,
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8
    },
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### cURL 示例

```bash
curl -X GET "https://api.sharestack.com/api/v1/articles?page=1&limit=10&category=frontend&sort=views&order=desc" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### 1.3 获取文章详情

**接口路径**: `GET /articles/{id}`

**描述**: 获取指定文章的详细信息。

#### 路径参数

- `id`: 文章ID

#### 查询参数

```
include_content: 是否包含完整内容，默认true
track_view: 是否记录阅读量，默认true
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "article_id": 12345,
    "title": "如何构建高性能的React应用",
    "content": "在现代Web开发中，React已经成为最受欢迎的前端框架之一...",
    "excerpt": "本文介绍了构建高性能React应用的最佳实践",
    "cover_image": "https://cdn.sharestack.com/covers/12345.jpg",
    "tags": ["React", "性能优化", "前端"],
    "category": {
      "id": 1,
      "name": "前端开发",
      "slug": "frontend"
    },
    "visibility": "public",
    "allow_comments": true,
    "status": "published",
    "seo_title": "高性能React应用构建指南",
    "seo_description": "学习如何构建高性能的React应用",
    "custom_url": "high-performance-react-app",
    "author": {
      "user_id": 67890,
      "username": "react_master",
      "display_name": "React大师",
      "avatar": "https://cdn.sharestack.com/avatars/67890.jpg",
      "bio": "资深前端工程师，React专家"
    },
    "stats": {
      "views_count": 1250,
      "likes_count": 89,
      "comments_count": 23,
      "bookmarks_count": 156,
      "shares_count": 34
    },
    "published_at": "2025-09-15T01:07:11Z",
    "created_at": "2025-09-15T01:07:11Z",
    "updated_at": "2025-09-15T01:07:11Z",
    "user_interactions": {
      "is_liked": false,
      "is_bookmarked": false,
      "is_subscribed_to_author": false
    },
    "reading_time": 8,
    "word_count": 2400
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.4 更新文章

**接口路径**: `PUT /articles/{id}`

**描述**: 更新指定文章的信息。

#### 路径参数

- `id`: 文章ID

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "title": "string",                  // 文章标题，可选
  "content": "string",                // 文章内容，可选
  "excerpt": "string",                // 文章摘要，可选
  "cover_image": "string",            // 封面图片URL，可选
  "tags": ["string"],                 // 标签数组，可选
  "category_id": "integer",           // 分类ID，可选
  "visibility": "string",             // 可见性，可选
  "allow_comments": "boolean",        // 是否允许评论，可选
  "seo_title": "string",              // SEO标题，可选
  "seo_description": "string",        // SEO描述，可选
  "custom_url": "string"              // 自定义URL别名，可选
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "文章更新成功",
    "article_id": 12345,
    "updated_fields": ["title", "content", "tags"],
    "updated_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.5 删除文章

**接口路径**: `DELETE /articles/{id}`

**描述**: 删除指定文章。

#### 路径参数

- `id`: 文章ID

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
    "message": "文章删除成功",
    "article_id": 12345,
    "deleted_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.6 发布文章

**接口路径**: `POST /articles/{id}/publish`

**描述**: 发布草稿文章。

#### 路径参数

- `id`: 文章ID

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 请求参数

```json
{
  "scheduled_at": "datetime"          // 定时发布时间，可选，默认立即发布
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "文章发布成功",
    "article_id": 12345,
    "status": "published",
    "published_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 1.7 取消发布文章

**接口路径**: `POST /articles/{id}/unpublish`

**描述**: 取消发布已发布的文章，转为草稿状态。

#### 路径参数

- `id`: 文章ID

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
    "message": "文章已转为草稿",
    "article_id": 12345,
    "status": "draft",
    "unpublished_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 2. 草稿和版本管理接口

### 2.1 获取草稿列表

**接口路径**: `GET /articles/drafts`

**描述**: 获取当前用户的草稿文章列表。

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 查询参数

```
page: 页码，默认1
limit: 每页数量，默认20
sort: 排序方式 (created_at|updated_at)
order: 排序方向 (asc|desc)，默认desc
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "drafts": [
      {
        "article_id": 12346,
        "title": "深入理解JavaScript闭包",
        "excerpt": "闭包是JavaScript中的重要概念...",
        "cover_image": null,
        "tags": ["JavaScript", "闭包"],
        "category": {
          "id": 1,
          "name": "前端开发"
        },
        "word_count": 1200,
        "created_at": "2025-09-14T15:30:00Z",
        "updated_at": "2025-09-15T01:07:11Z",
        "auto_save_count": 15
      }
    ]
  },
  "error": null,
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 8,
      "pages": 1
    },
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 2.2 自动保存草稿

**接口路径**: `POST /articles/{id}/autosave`

**描述**: 自动保存文章草稿，用于编辑器的自动保存功能。

#### 路径参数

- `id`: 文章ID

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "title": "string",                  // 文章标题，可选
  "content": "string",                // 文章内容，可选
  "excerpt": "string",                // 文章摘要，可选
  "tags": ["string"],                 // 标签数组，可选
  "category_id": "integer"            // 分类ID，可选
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "草稿自动保存成功",
    "article_id": 12346,
    "autosave_version": 16,
    "saved_at": "2025-09-15T01:07:11Z"
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
// 自动保存功能实现
let autoSaveTimer;
const AUTOSAVE_INTERVAL = 30000; // 30秒自动保存

const autoSave = async (articleId, content) => {
  const token = localStorage.getItem('access_token');

  try {
    const response = await fetch(`/api/v1/articles/${articleId}/autosave`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(content)
    });

    const result = await response.json();
    if (result.success) {
      console.log('草稿已自动保存');
    }
  } catch (error) {
    console.error('自动保存失败:', error);
  }
};

// 编辑器内容变化时重置定时器
const onContentChange = (articleId, content) => {
  clearTimeout(autoSaveTimer);
  autoSaveTimer = setTimeout(() => {
    autoSave(articleId, content);
  }, AUTOSAVE_INTERVAL);
};
```

---

### 2.3 获取版本历史

**接口路径**: `GET /articles/{id}/versions`

**描述**: 获取文章的版本历史记录。

#### 路径参数

- `id`: 文章ID

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 查询参数

```
page: 页码，默认1
limit: 每页数量，默认10
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "versions": [
      {
        "version_id": 1001,
        "version_number": 3,
        "title": "如何构建高性能的React应用",
        "content_preview": "在现代Web开发中，React已经成为最受欢迎的前端框架之一...",
        "changes_summary": "更新了性能优化章节，添加了新的代码示例",
        "word_count": 2400,
        "is_current": true,
        "created_at": "2025-09-15T01:07:11Z",
        "author": {
          "user_id": 67890,
          "username": "react_master",
          "display_name": "React大师"
        }
      },
      {
        "version_id": 1000,
        "version_number": 2,
        "title": "如何构建高性能的React应用",
        "content_preview": "在现代Web开发中，React已经成为最受欢迎的前端框架...",
        "changes_summary": "修复了代码示例中的错误",
        "word_count": 2200,
        "is_current": false,
        "created_at": "2025-09-14T18:30:00Z",
        "author": {
          "user_id": 67890,
          "username": "react_master",
          "display_name": "React大师"
        }
      }
    ]
  },
  "error": null,
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 3,
      "pages": 1
    },
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 2.4 恢复版本

**接口路径**: `POST /articles/{id}/restore`

**描述**: 恢复文章到指定的历史版本。

#### 路径参数

- `id`: 文章ID

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "version_id": "integer"             // 要恢复的版本ID
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "版本恢复成功",
    "article_id": 12345,
    "restored_version": 2,
    "new_version": 4,
    "restored_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 3. 媒体文件管理接口

### 3.1 图片上传

**接口路径**: `POST /uploads/image`

**描述**: 上传图片文件，支持多种格式和自动处理。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

#### 请求参数

```
image: File                         // 图片文件，必填
alt_text: string                    // 图片描述文本，可选
optimize: boolean                   // 是否自动优化，默认true
generate_thumbnails: boolean        // 是否生成缩略图，默认true
```

#### 文件限制

- **支持格式**: JPEG, PNG, GIF, WebP
- **文件大小**: 最大10MB
- **图片尺寸**: 最大4096x4096像素

#### 响应示例

**成功响应 (201)**:
```json
{
  "success": true,
  "data": {
    "file_id": "img_12345",
    "filename": "react-performance-chart.png",
    "original_url": "https://cdn.sharestack.com/images/react-performance-chart.png",
    "optimized_url": "https://cdn.sharestack.com/images/optimized/react-performance-chart.webp",
    "thumbnails": {
      "small": "https://cdn.sharestack.com/images/thumbs/small/react-performance-chart.webp",
      "medium": "https://cdn.sharestack.com/images/thumbs/medium/react-performance-chart.webp",
      "large": "https://cdn.sharestack.com/images/thumbs/large/react-performance-chart.webp"
    },
    "alt_text": "React应用性能优化对比图表",
    "file_size": 245760,
    "dimensions": {
      "width": 1200,
      "height": 800
    },
    "format": "png",
    "uploaded_at": "2025-09-15T01:07:11Z"
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
curl -X POST "https://api.sharestack.com/api/v1/uploads/image" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -F "image=@/path/to/image.png" \
  -F "alt_text=React应用性能优化对比图表" \
  -F "optimize=true" \
  -F "generate_thumbnails=true"
```

#### JavaScript 示例

```javascript
const uploadImage = async (file, altText = '') => {
  const token = localStorage.getItem('access_token');
  const formData = new FormData();
  formData.append('image', file);
  formData.append('alt_text', altText);
  formData.append('optimize', 'true');
  formData.append('generate_thumbnails', 'true');

  const response = await fetch('/api/v1/uploads/image', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`
    },
    body: formData
  });

  const result = await response.json();
  return result;
};

// 使用示例 - 拖拽上传
const dropZone = document.getElementById('drop-zone');

dropZone.addEventListener('drop', async (e) => {
  e.preventDefault();
  const files = e.dataTransfer.files;

  for (const file of files) {
    if (file.type.startsWith('image/')) {
      const result = await uploadImage(file, '用户上传的图片');
      if (result.success) {
        console.log('图片上传成功:', result.data.original_url);
      }
    }
  }
});
```

---

### 3.2 视频上传

**接口路径**: `POST /uploads/video`

**描述**: 上传视频文件，支持多种格式和转码处理。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

#### 请求参数

```
video: File                         // 视频文件，必填
title: string                       // 视频标题，可选
description: string                 // 视频描述，可选
transcode: boolean                  // 是否转码，默认true
generate_thumbnail: boolean         // 是否生成封面，默认true
```

#### 文件限制

- **支持格式**: MP4, MOV, AVI, MKV, WebM
- **文件大小**: 最大500MB
- **视频时长**: 最大60分钟
- **分辨率**: 最大1920x1080

#### 响应示例

**成功响应 (202)**:
```json
{
  "success": true,
  "data": {
    "file_id": "vid_12345",
    "filename": "react-tutorial.mp4",
    "upload_status": "processing",
    "original_url": "https://cdn.sharestack.com/videos/original/react-tutorial.mp4",
    "title": "React基础教程",
    "description": "从零开始学习React框架",
    "file_size": 52428800,
    "duration": 1200,
    "format": "mp4",
    "processing_job_id": "job_67890",
    "estimated_completion": "2025-09-15T01:15:00Z",
    "uploaded_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

#### 处理完成后的Webhook通知

```json
{
  "event": "video.processing.completed",
  "data": {
    "file_id": "vid_12345",
    "status": "completed",
    "transcoded_urls": {
      "720p": "https://cdn.sharestack.com/videos/720p/react-tutorial.mp4",
      "480p": "https://cdn.sharestack.com/videos/480p/react-tutorial.mp4",
      "360p": "https://cdn.sharestack.com/videos/360p/react-tutorial.mp4"
    },
    "thumbnail_url": "https://cdn.sharestack.com/videos/thumbs/react-tutorial.jpg",
    "preview_gif": "https://cdn.sharestack.com/videos/previews/react-tutorial.gif"
  }
}
```

---

### 3.3 音频上传

**接口路径**: `POST /uploads/audio`

**描述**: 上传音频文件，支持多种格式和转码处理。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: multipart/form-data
```

#### 请求参数

```
audio: File                         // 音频文件，必填
title: string                       // 音频标题，可选
artist: string                      // 艺术家/作者，可选
transcode: boolean                  // 是否转码，默认true
generate_waveform: boolean          // 是否生成波形图，默认true
```

#### 文件限制

- **支持格式**: MP3, WAV, M4A, FLAC, OGG
- **文件大小**: 最大100MB
- **音频时长**: 最大120分钟

#### 响应示例

**成功响应 (202)**:
```json
{
  "success": true,
  "data": {
    "file_id": "aud_12345",
    "filename": "podcast-episode-1.mp3",
    "upload_status": "processing",
    "original_url": "https://cdn.sharestack.com/audio/original/podcast-episode-1.mp3",
    "title": "前端开发播客第一期",
    "artist": "前端开发者",
    "file_size": 15728640,
    "duration": 1800,
    "format": "mp3",
    "bitrate": 128,
    "processing_job_id": "job_67891",
    "estimated_completion": "2025-09-15T01:10:00Z",
    "uploaded_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 3.4 获取文件信息

**接口路径**: `GET /uploads/{id}`

**描述**: 获取已上传文件的详细信息。

#### 路径参数

- `id`: 文件ID

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
    "file_id": "img_12345",
    "filename": "react-performance-chart.png",
    "file_type": "image",
    "original_url": "https://cdn.sharestack.com/images/react-performance-chart.png",
    "optimized_url": "https://cdn.sharestack.com/images/optimized/react-performance-chart.webp",
    "thumbnails": {
      "small": "https://cdn.sharestack.com/images/thumbs/small/react-performance-chart.webp",
      "medium": "https://cdn.sharestack.com/images/thumbs/medium/react-performance-chart.webp",
      "large": "https://cdn.sharestack.com/images/thumbs/large/react-performance-chart.webp"
    },
    "alt_text": "React应用性能优化对比图表",
    "file_size": 245760,
    "dimensions": {
      "width": 1200,
      "height": 800
    },
    "format": "png",
    "usage_count": 3,
    "used_in_articles": [
      {
        "article_id": 12345,
        "title": "如何构建高性能的React应用"
      }
    ],
    "uploaded_at": "2025-09-15T01:07:11Z",
    "updated_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 3.5 删除文件

**接口路径**: `DELETE /uploads/{id}`

**描述**: 删除指定的上传文件。

#### 路径参数

- `id`: 文件ID

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
    "message": "文件删除成功",
    "file_id": "img_12345",
    "deleted_urls": [
      "https://cdn.sharestack.com/images/react-performance-chart.png",
      "https://cdn.sharestack.com/images/optimized/react-performance-chart.webp"
    ],
    "deleted_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 4. 分类标签管理接口

### 4.1 获取分类列表

**接口路径**: `GET /categories`

**描述**: 获取所有内容分类列表。

#### 查询参数

```
parent_id: 父分类ID，获取子分类
include_count: 是否包含文章数量，默认false
status: 分类状态筛选 (active|inactive)
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "categories": [
      {
        "id": 1,
        "name": "前端开发",
        "slug": "frontend",
        "description": "前端技术相关内容",
        "color": "#3498db",
        "icon": "code",
        "parent_id": null,
        "order": 1,
        "status": "active",
        "articles_count": 45,
        "children": [
          {
            "id": 11,
            "name": "React",
            "slug": "react",
            "description": "React框架相关内容",
            "color": "#61dafb",
            "parent_id": 1,
            "articles_count": 20
          },
          {
            "id": 12,
            "name": "Vue.js",
            "slug": "vue",
            "description": "Vue.js框架相关内容",
            "color": "#4fc08d",
            "parent_id": 1,
            "articles_count": 15
          }
        ],
        "created_at": "2025-01-01T00:00:00Z",
        "updated_at": "2025-09-15T01:07:11Z"
      }
    ]
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 4.2 创建分类

**接口路径**: `POST /categories`

**描述**: 创建新的内容分类。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "name": "string",                   // 分类名称，必填，1-50字符
  "slug": "string",                   // URL别名，可选，自动生成
  "description": "string",            // 分类描述，可选，0-200字符
  "color": "string",                  // 分类颜色，可选，十六进制格式
  "icon": "string",                   // 分类图标，可选
  "parent_id": "integer",             // 父分类ID，可选
  "order": "integer"                  // 排序权重，可选，默认0
}
```

#### 响应示例

**成功响应 (201)**:
```json
{
  "success": true,
  "data": {
    "id": 13,
    "name": "Node.js",
    "slug": "nodejs",
    "description": "Node.js后端开发相关内容",
    "color": "#68a063",
    "icon": "server",
    "parent_id": null,
    "order": 3,
    "status": "active",
    "articles_count": 0,
    "created_at": "2025-09-15T01:07:11Z",
    "updated_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 4.3 更新分类

**接口路径**: `PUT /categories/{id}`

**描述**: 更新指定分类的信息。

#### 路径参数

- `id`: 分类ID

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "name": "string",                   // 分类名称，可选
  "description": "string",            // 分类描述，可选
  "color": "string",                  // 分类颜色，可选
  "icon": "string",                   // 分类图标，可选
  "parent_id": "integer",             // 父分类ID，可选
  "order": "integer",                 // 排序权重，可选
  "status": "string"                  // 分类状态，可选 (active|inactive)
}
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "分类更新成功",
    "category_id": 13,
    "updated_fields": ["description", "color"],
    "updated_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 4.4 删除分类

**接口路径**: `DELETE /categories/{id}`

**描述**: 删除指定分类。注意：删除分类时需要处理该分类下的文章。

#### 路径参数

- `id`: 分类ID

#### 请求头

```
Authorization: Bearer <access_token>
```

#### 查询参数

```
action: 删除策略 (move|orphan)
target_category_id: 目标分类ID（当action=move时必填）
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "message": "分类删除成功",
    "category_id": 13,
    "affected_articles": 5,
    "action_taken": "moved_to_category_1",
    "deleted_at": "2025-09-15T01:07:11Z"
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 4.5 获取标签列表

**接口路径**: `GET /tags`

**描述**: 获取所有标签列表。

#### 查询参数

```
search: 搜索关键词
popular: 是否只返回热门标签 (true|false)
limit: 返回数量限制，默认100
min_usage: 最小使用次数筛选
```

#### 响应示例

**成功响应 (200)**:
```json
{
  "success": true,
  "data": {
    "tags": [
      {
        "id": 1,
        "name": "React",
        "slug": "react",
        "description": "React JavaScript库",
        "color": "#61dafb",
        "usage_count": 120,
        "trending_score": 95,
        "created_at": "2025-01-01T00:00:00Z",
        "last_used_at": "2025-09-15T01:07:11Z"
      },
      {
        "id": 2,
        "name": "JavaScript",
        "slug": "javascript",
        "description": "JavaScript编程语言",
        "color": "#f7df1e",
        "usage_count": 200,
        "trending_score": 88,
        "created_at": "2025-01-01T00:00:00Z",
        "last_used_at": "2025-09-15T01:07:11Z"
      }
    ]
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

### 4.6 创建标签

**接口路径**: `POST /tags`

**描述**: 创建新标签。

#### 请求头

```
Authorization: Bearer <access_token>
Content-Type: application/json
```

#### 请求参数

```json
{
  "name": "string",                   // 标签名称，必填，1-30字符
  "description": "string",            // 标签描述，可选，0-100字符
  "color": "string"                   // 标签颜色，可选，十六进制格式
}
```

#### 响应示例

**成功响应 (201)**:
```json
{
  "success": true,
  "data": {
    "id": 101,
    "name": "TypeScript",
    "slug": "typescript",
    "description": "TypeScript编程语言",
    "color": "#3178c6",
    "usage_count": 0,
    "trending_score": 0,
    "created_at": "2025-09-15T01:07:11Z",
    "last_used_at": null
  },
  "error": null,
  "meta": {
    "version": "v1",
    "timestamp": "2025-09-15T01:07:11Z"
  }
}
```

---

## 文件上传流程说明

### 分片上传大文件

对于大文件（>100MB），建议使用分片上传机制：

#### 1. 初始化分片上传

```bash
curl -X POST "https://api.sharestack.com/api/v1/uploads/multipart/init" \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "large-video.mp4",
    "file_size": 524288000,
    "file_type": "video/mp4",
    "chunk_size": 10485760
  }'
```

#### 2. 上传分片

```bash
curl -X PUT "https://api.sharestack.com/api/v1/uploads/multipart/upload" \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/octet-stream" \
  -H "X-Upload-ID: upload_12345" \
  -H "X-Chunk-Number: 1" \
  --data-binary @chunk_001.bin
```

#### 3. 完成分片上传

```bash
curl -X POST "https://api.sharestack.com/api/v1/uploads/multipart/complete" \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "upload_id": "upload_12345",
    "chunks": [
      {"chunk_number": 1, "etag": "abc123"},
      {"chunk_number": 2, "etag": "def456"}
    ]
  }'
```

### 富文本编辑器集成

#### 图片粘贴上传

```javascript
const handlePaste = async (event) => {
  const items = event.clipboardData.items;

  for (const item of items) {
    if (item.type.startsWith('image/')) {
      const file = item.getAsFile();
      const result = await uploadImage(file, '粘贴的图片');

      if (result.success) {
        // 插入图片到编辑器
        const imageUrl = result.data.optimized_url;
        const altText = result.data.alt_text;
        insertImageToEditor(imageUrl, altText);
      }
    }
  }
};

// 绑定粘贴事件
editor.addEventListener('paste', handlePaste);
```

#### 拖拽上传

```javascript
const handleDrop = async (event) => {
  event.preventDefault();
  const files = event.dataTransfer.files;

  for (const file of files) {
    let uploadResult;

    if (file.type.startsWith('image/')) {
      uploadResult = await uploadImage(file);
    } else if (file.type.startsWith('video/')) {
      uploadResult = await uploadVideo(file);
    } else if (file.type.startsWith('audio/')) {
      uploadResult = await uploadAudio(file);
    }

    if (uploadResult && uploadResult.success) {
      insertMediaToEditor(uploadResult.data);
    }
  }
};
```

---

## 媒体文件安全控制

### 访问权限控制

所有媒体文件的访问都通过CDN和签名URL进行控制：

#### 私有内容访问

```javascript
// 获取私有内容的临时访问URL
const getPrivateMediaUrl = async (fileId) => {
  const token = localStorage.getItem('access_token');

  const response = await fetch(`/api/v1/uploads/${fileId}/access-url`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      expires_in: 3600  // 1小时有效期
    })
  });

  const result = await response.json();
  return result.data.signed_url;
};
```

### 文件类型验证

服务端对所有上传文件进行严格的类型验证：

1. **MIME类型检查**: 验证文件的真实MIME类型
2. **文件头验证**: 检查文件头魔数
3. **病毒扫描**: 对上传文件进行安全扫描
4. **内容审核**: 对图片和视频进行内容审核

### 存储安全

1. **分布式存储**: 文件存储在多个地理位置
2. **加密存储**: 敏感文件进行加密存储
3. **备份策略**: 定期备份重要文件
4. **访问日志**: 记录所有文件访问日志

---

## SEO优化接口

### 自定义URL设置

所有文章都支持自定义URL别名，用于SEO优化：

```json
{
  "custom_url": "high-performance-react-app",
  "seo_title": "高性能React应用构建指南 - ShareStack",
  "seo_description": "学习如何构建高性能的React应用，包括代码分割、懒加载、性能监控等最佳实践",
  "seo_keywords": ["React", "性能优化", "前端开发", "JavaScript"],
  "canonical_url": "https://sharestack.com/articles/high-performance-react-app",
  "robots": "index,follow",
  "og_image": "https://cdn.sharestack.com/og/high-performance-react-app.jpg"
}
```

### 结构化数据

API自动生成结构化数据用于搜索引擎优化：

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "如何构建高性能的React应用",
  "author": {
    "@type": "Person",
    "name": "React大师",
    "url": "https://sharestack.com/users/react_master"
  },
  "datePublished": "2025-09-15T01:07:11Z",
  "dateModified": "2025-09-15T01:07:11Z",
  "image": "https://cdn.sharestack.com/covers/12345.jpg",
  "publisher": {
    "@type": "Organization",
    "name": "ShareStack",
    "logo": "https://sharestack.com/logo.png"
  }
}
```

---

## 最佳实践

### 内容创作工作流

1. **创建草稿**: 使用 `POST /articles` 创建初始草稿
2. **自动保存**: 定期调用 `POST /articles/{id}/autosave` 保存编辑内容
3. **媒体上传**: 使用相应的上传接口添加图片、视频、音频
4. **预览检查**: 获取草稿内容进行预览
5. **发布文章**: 调用 `POST /articles/{id}/publish` 正式发布

### 性能优化建议

1. **图片优化**:
   - 启用自动优化功能
   - 生成多种尺寸的缩略图
   - 使用WebP格式提高加载速度

2. **内容分页**:
   - 文章列表使用分页避免一次加载过多数据
   - 推荐每页20-50篇文章

3. **缓存策略**:
   - 文章内容设置适当的缓存时间
   - 媒体文件使用CDN加速

### 安全考虑

1. **内容验证**:
   - 所有用户输入进行XSS过滤
   - HTML内容进行安全清理
   - 防止SQL注入攻击

2. **权限控制**:
   - 创作者只能管理自己的内容
   - 管理员拥有全部内容管理权限
   - 订阅者根据订阅状态访问付费内容

3. **审核机制**:
   - 敏感内容自动标记审核
   - 举报处理机制
   - 违规内容处理流程

---

## OpenAPI 3.0 规范

```yaml
openapi: 3.0.3
info:
  title: ShareStack 内容管理 API
  description: ShareStack 知识付费平台内容管理模块 API 接口
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
    Article:
      type: object
      properties:
        article_id:
          type: integer
          example: 12345
        title:
          type: string
          example: "如何构建高性能的React应用"
        content:
          type: string
          example: "在现代Web开发中..."
        excerpt:
          type: string
          example: "本文介绍了构建高性能React应用的最佳实践"
        cover_image:
          type: string
          format: uri
          example: "https://cdn.sharestack.com/covers/12345.jpg"
        tags:
          type: array
          items:
            type: string
          example: ["React", "性能优化", "前端"]
        visibility:
          type: string
          enum: [public, subscribers, paid]
        status:
          type: string
          enum: [draft, published, archived]
        published_at:
          type: string
          format: date-time
          example: "2025-09-15T01:07:11Z"

    MediaFile:
      type: object
      properties:
        file_id:
          type: string
          example: "img_12345"
        filename:
          type: string
          example: "react-chart.png"
        file_type:
          type: string
          enum: [image, video, audio]
        original_url:
          type: string
          format: uri
          example: "https://cdn.sharestack.com/images/react-chart.png"
        file_size:
          type: integer
          example: 245760
        uploaded_at:
          type: string
          format: date-time

    Category:
      type: object
      properties:
        id:
          type: integer
          example: 1
        name:
          type: string
          example: "前端开发"
        slug:
          type: string
          example: "frontend"
        description:
          type: string
          example: "前端技术相关内容"
        color:
          type: string
          example: "#3498db"
        parent_id:
          type: integer
          nullable: true
        articles_count:
          type: integer
          example: 45

    Tag:
      type: object
      properties:
        id:
          type: integer
          example: 1
        name:
          type: string
          example: "React"
        slug:
          type: string
          example: "react"
        description:
          type: string
          example: "React JavaScript库"
        usage_count:
          type: integer
          example: 120

security:
  - bearerAuth: []

paths:
  /articles:
    post:
      summary: 创建文章
      tags: [文章管理]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [title, content]
              properties:
                title:
                  type: string
                  minLength: 1
                  maxLength: 200
                content:
                  type: string
                  minLength: 1
                excerpt:
                  type: string
                  maxLength: 500
                tags:
                  type: array
                  items:
                    type: string
                  maxItems: 10
                category_id:
                  type: integer
                visibility:
                  type: string
                  enum: [public, subscribers, paid]
                  default: public
                status:
                  type: string
                  enum: [draft, published]
                  default: draft
      responses:
        '201':
          description: 文章创建成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/Article'

    get:
      summary: 获取文章列表
      tags: [文章管理]
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
        - name: status
          in: query
          schema:
            type: string
            enum: [draft, published, archived]
        - name: category
          in: query
          schema:
            type: string
        - name: tags
          in: query
          schema:
            type: string
        - name: sort
          in: query
          schema:
            type: string
            enum: [created_at, updated_at, published_at, views, likes]
            default: published_at
        - name: order
          in: query
          schema:
            type: string
            enum: [asc, desc]
            default: desc
      responses:
        '200':
          description: 文章列表获取成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    type: object
                    properties:
                      articles:
                        type: array
                        items:
                          $ref: '#/components/schemas/Article'

  /uploads/image:
    post:
      summary: 上传图片
      tags: [媒体管理]
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              required: [image]
              properties:
                image:
                  type: string
                  format: binary
                alt_text:
                  type: string
                optimize:
                  type: boolean
                  default: true
                generate_thumbnails:
                  type: boolean
                  default: true
      responses:
        '201':
          description: 图片上传成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                    example: true
                  data:
                    $ref: '#/components/schemas/MediaFile'
```

---

## 变更日志

### v1.0.0 (2025-09-15)
- 初始版本发布
- 完成文章CRUD接口设计 (7个接口)
- 完成草稿和版本管理接口设计 (4个接口)
- 完成媒体文件管理接口设计 (5个接口)
- 完成分类标签管理接口设计 (6个接口)
- 添加分片上传大文件支持
- 实现富文本编辑器集成方案
- 提供SEO优化和安全控制机制
- 包含OpenAPI 3.0规范定义

---

**文档版本**: v1.0.0
**创建时间**: 2025-09-15
**最后更新**: 2025-09-15
**负责团队**: 后端开发团队

*本API文档基于ShareStack知识付费平台PRD需求制定，为内容管理模块提供完整的接口规范和实施指南。*