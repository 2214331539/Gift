# API 设计

## 1. API 目标

API 需要支撑用户端、管理端和后续移动端复用。MVP 推荐使用 REST 风格接口，配合统一错误结构、统一鉴权和 Zod 校验。

## 2. 通用约定

### 2.1 基础路径

```text
/api/v1
```

### 2.2 返回格式

成功：

```json
{
  "ok": true,
  "data": {},
  "meta": {}
}
```

失败：

```json
{
  "ok": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求参数不正确",
    "details": {}
  }
}
```

### 2.3 分页格式

请求：

```text
?page=1&pageSize=20
```

响应：

```json
{
  "ok": true,
  "data": [],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 128,
    "hasNext": true
  }
}
```

后续内容流可以改为 cursor：

```text
?cursor=2026-05-12T10:00:00.000Z_xxx&limit=20
```

### 2.4 鉴权

- 读公开内容：不需要登录。
- 写操作：需要登录。
- 管理接口：需要 `moderator` 或 `admin`。
- 用户只能修改自己的内容。

### 2.5 通用错误码

| 错误码 | HTTP | 说明 |
| --- | --- | --- |
| UNAUTHORIZED | 401 | 未登录 |
| FORBIDDEN | 403 | 无权限 |
| NOT_FOUND | 404 | 资源不存在 |
| VALIDATION_ERROR | 422 | 参数错误 |
| CONFLICT | 409 | 状态冲突或重复操作 |
| RATE_LIMITED | 429 | 请求过快 |
| INTERNAL_ERROR | 500 | 服务端错误 |

## 3. Auth API

### 3.1 注册

```http
POST /api/v1/auth/register
```

请求：

```json
{
  "email": "user@example.com",
  "password": "password",
  "nickname": "小礼物"
}
```

响应：

```json
{
  "ok": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "nickname": "小礼物",
      "role": "user"
    }
  }
}
```

### 3.2 当前用户

```http
GET /api/v1/auth/me
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "email": "user@example.com",
    "nickname": "小礼物",
    "avatarUrl": null,
    "role": "user"
  }
}
```

## 4. 礼物故事 API

### 4.1 礼物列表

```http
GET /api/v1/stories
```

查询参数：

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| q | string | 关键词 |
| recipient | string | 送给谁 |
| occasion | string | 场景 |
| relationshipStage | string | 关系阶段 |
| budget | string | 预算区间 |
| preparationTime | string | 准备时间 |
| giftType | string | 礼物类型 |
| interests | string[] | 兴趣标签 |
| emotions | string[] | 情绪标签 |
| isFailureCase | boolean | 是否失败避坑 |
| sort | string | relevant, latest, saved, forked, low_budget |
| page | number | 页码 |
| pageSize | number | 每页数量 |

响应：

```json
{
  "ok": true,
  "data": [
    {
      "id": "uuid",
      "slug": "restart-life-gift-box",
      "title": "给考研结束的女朋友做了一个重启生活礼盒",
      "summary": "她拆到最后一张卡片的时候哭了。",
      "coverImageUrl": "https://cdn.example.com/image.webp",
      "recipientType": "女朋友",
      "occasion": "考研结束",
      "budgetRange": "100-300",
      "preparationTime": "2 天",
      "giftType": "手工礼盒",
      "saveCount": 102,
      "reactionCount": 328,
      "forkCount": 17,
      "publishedAt": "2026-05-12T08:00:00.000Z"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 128,
    "hasNext": true
  }
}
```

业务规则：

- 只返回 `approved + public` 内容。
- 如果用户登录，可以附加 `isSavedByMe`、`isReactedByMe`。

### 4.2 礼物详情

```http
GET /api/v1/stories/:slug
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "slug": "restart-life-gift-box",
    "title": "给考研结束的女朋友做了一个重启生活礼盒",
    "summary": "她拆到最后一张卡片的时候哭了。",
    "author": {
      "id": "uuid",
      "nickname": "匿名用户",
      "avatarUrl": null,
      "isAnonymous": true
    },
    "images": [
      {
        "id": "uuid",
        "url": "https://cdn.example.com/image.webp",
        "thumbnailUrl": "https://cdn.example.com/thumb.webp",
        "altText": "礼盒照片"
      }
    ],
    "recipientType": "女朋友",
    "relationshipStage": "1-3 年",
    "occasion": "考研结束",
    "budgetRange": "100-300",
    "preparationTime": "2 天",
    "giftType": "手工礼盒",
    "giftItems": [
      "一本手写小册子",
      "一张周末出游计划",
      "一盒她喜欢的茶"
    ],
    "description": "整体礼物描述",
    "reason": "她考研结束后整个人很空，我想让她感觉接下来的生活也值得期待。",
    "preparationProcess": "准备过程",
    "reaction": "她拆到最后一张卡片的时候哭了。",
    "tips": "手写内容比礼盒本身更重要。",
    "suitableFor": "对方刚结束一段高压时期",
    "notSuitableFor": "对方不喜欢仪式感时不适合",
    "tags": [
      { "id": "uuid", "type": "occasion", "name": "考研结束", "slug": "exam-finished" }
    ],
    "stats": {
      "viewCount": 1200,
      "saveCount": 102,
      "reactionCount": 328,
      "commentCount": 18,
      "forkCount": 17,
      "thanksCount": 6
    },
    "viewerState": {
      "isSaved": false,
      "isReacted": false,
      "hasThanked": false
    }
  }
}
```

### 4.3 创建草稿

```http
POST /api/v1/stories
```

请求：

```json
{
  "title": "给考研结束的女朋友做了一个重启生活礼盒",
  "recipientType": "女朋友",
  "relationshipStage": "1-3 年",
  "occasion": "考研结束",
  "budgetRange": "100-300",
  "preparationTime": "2 天",
  "giftType": "手工礼盒",
  "giftItems": ["一本手写小册子", "一张周末出游计划"],
  "reason": "她考研结束后整个人很空。",
  "description": "整体礼物描述",
  "preparationProcess": "准备过程",
  "reaction": "她拆到最后一张卡片的时候哭了。",
  "tips": "手写内容比礼盒本身更重要。",
  "suitableFor": "刚结束高压时期的人",
  "notSuitableFor": "不喜欢仪式感的人",
  "imageIds": ["uuid"],
  "tagIds": ["uuid"],
  "isAnonymous": true,
  "hideExactBudget": true,
  "isFailureCase": false
}
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "status": "draft"
  }
}
```

### 4.4 提交审核

```http
POST /api/v1/stories/:id/submit
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "status": "pending"
  }
}
```

校验：

- 必须是作者。
- 必填字段完整。
- 至少一张图片。
- 至少包含送给谁、场景、预算标签。

### 4.5 更新故事

```http
PATCH /api/v1/stories/:id
```

规则：

- 作者可更新自己的草稿或被驳回内容。
- 已公开内容更新后可进入重新审核流程。

### 4.6 删除故事

```http
DELETE /api/v1/stories/:id
```

规则：

- 作者可删除自己的内容。
- 管理员可删除违规内容。
- 默认软删除。

## 5. 图片上传 API

### 5.1 获取上传凭证

```http
POST /api/v1/uploads/presign
```

请求：

```json
{
  "filename": "gift.jpg",
  "mimeType": "image/jpeg",
  "sizeBytes": 2048000
}
```

响应：

```json
{
  "ok": true,
  "data": {
    "assetId": "uuid",
    "uploadUrl": "https://storage.example.com/...",
    "headers": {
      "Content-Type": "image/jpeg"
    },
    "publicUrl": "https://cdn.example.com/gift.jpg"
  }
}
```

### 5.2 确认上传

```http
POST /api/v1/uploads/:assetId/complete
```

请求：

```json
{
  "width": 1200,
  "height": 900,
  "privacyChecked": true
}
```

## 6. 标签 API

### 6.1 标签列表

```http
GET /api/v1/tags
```

参数：

```text
?type=occasion&active=true
```

响应：

```json
{
  "ok": true,
  "data": [
    {
      "id": "uuid",
      "type": "occasion",
      "name": "生日",
      "slug": "birthday",
      "usageCount": 120,
      "children": []
    }
  ]
}
```

### 6.2 热门标签

```http
GET /api/v1/tags/hot
```

用于首页快捷入口和搜索页推荐。

## 7. 收藏 API

### 7.1 收藏故事

```http
POST /api/v1/stories/:id/save
```

响应：

```json
{
  "ok": true,
  "data": {
    "isSaved": true,
    "saveCount": 103
  }
}
```

### 7.2 取消收藏

```http
DELETE /api/v1/stories/:id/save
```

### 7.3 我的收藏

```http
GET /api/v1/me/collections/default/items
```

## 8. 评论 API

### 8.1 评论列表

```http
GET /api/v1/stories/:id/comments?page=1&pageSize=20
```

### 8.2 创建评论

```http
POST /api/v1/stories/:id/comments
```

请求：

```json
{
  "content": "这个思路很适合我，想参考一下。",
  "parentId": null
}
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "content": "这个思路很适合我，想参考一下。",
    "status": "visible",
    "createdAt": "2026-05-12T08:00:00.000Z"
  }
}
```

### 8.3 删除评论

```http
DELETE /api/v1/comments/:id
```

## 9. 互动 API

### 9.1 很用心

```http
POST /api/v1/stories/:id/reactions/thoughtful
```

取消：

```http
DELETE /api/v1/stories/:id/reactions/thoughtful
```

### 9.2 感谢作者

```http
POST /api/v1/stories/:id/reactions/thanks
```

### 9.3 我参考了这个

MVP 记录事件：

```http
POST /api/v1/stories/:id/forks
```

请求：

```json
{
  "modificationDescription": "我想改成语音卡片版本"
}
```

P1 支持关联新故事：

```json
{
  "newStoryId": "uuid",
  "modificationDescription": "我改成了 30 封邮件，每天定时发送。"
}
```

## 10. 举报 API

### 10.1 提交举报

```http
POST /api/v1/reports
```

请求：

```json
{
  "targetType": "story",
  "targetId": "uuid",
  "reason": "privacy",
  "description": "图片中可能包含真实姓名和地址。"
}
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "status": "pending"
  }
}
```

## 11. 用户 API

### 11.1 用户主页

```http
GET /api/v1/users/:id
```

响应：

```json
{
  "ok": true,
  "data": {
    "id": "uuid",
    "nickname": "礼物记录者",
    "avatarUrl": null,
    "bio": "喜欢记录用心的小事",
    "stats": {
      "publishedCount": 12,
      "helpedCount": 86,
      "thanksReceivedCount": 24
    }
  }
}
```

### 11.2 用户发布列表

```http
GET /api/v1/users/:id/stories
```

### 11.3 我的发布

```http
GET /api/v1/me/stories?status=draft,pending,rejected,approved
```

## 12. 首页 API

### 12.1 首页聚合

```http
GET /api/v1/home
```

响应：

```json
{
  "ok": true,
  "data": {
    "featured": [],
    "hotScenarios": [],
    "lowBudget": [],
    "latest": [],
    "failureCases": [],
    "festivalCountdown": {
      "name": "七夕",
      "daysLeft": 23,
      "storyCount": 1284
    }
  }
}
```

## 13. 管理 API

管理接口统一前缀：

```text
/api/v1/admin
```

### 13.1 内容审核列表

```http
GET /api/v1/admin/stories?status=pending&page=1&pageSize=20
```

### 13.2 审核通过

```http
POST /api/v1/admin/stories/:id/approve
```

请求：

```json
{
  "note": "内容真实，字段完整。"
}
```

### 13.3 审核驳回

```http
POST /api/v1/admin/stories/:id/reject
```

请求：

```json
{
  "reason": "图片中包含明显隐私信息，请处理后重新提交。"
}
```

### 13.4 隐藏内容

```http
POST /api/v1/admin/stories/:id/hide
```

### 13.5 举报列表

```http
GET /api/v1/admin/reports?status=pending
```

### 13.6 处理举报

```http
POST /api/v1/admin/reports/:id/resolve
```

请求：

```json
{
  "action": "hide_target",
  "resolution": "内容包含隐私信息，已隐藏并通知作者。"
}
```

### 13.7 标签管理

```http
POST /api/v1/admin/tags
PATCH /api/v1/admin/tags/:id
POST /api/v1/admin/tags/:id/merge
DELETE /api/v1/admin/tags/:id
```

### 13.8 精选内容

```http
GET /api/v1/admin/featured
POST /api/v1/admin/featured
PATCH /api/v1/admin/featured/:id
DELETE /api/v1/admin/featured/:id
```

## 14. 埋点 API

### 14.1 上报事件

```http
POST /api/v1/events
```

请求：

```json
{
  "eventName": "story_save",
  "targetType": "story",
  "targetId": "uuid",
  "properties": {
    "source": "detail"
  }
}
```

说明：

- 关键业务事件可服务端自动记录，不完全依赖前端上报。
- 埋点接口需要限流。

## 15. API 验收标准

- 所有写接口都校验登录态。
- 管理接口校验角色。
- 用户不能修改他人的内容。
- 列表接口不返回待审核、隐藏、删除内容。
- 参数错误返回 422 和字段级详情。
- 重复收藏、重复点赞返回幂等结果或 409，行为需一致。
- 审核接口必须写入审核日志。
- 上传接口限制文件类型、大小和数量。
- API 响应结构统一。
