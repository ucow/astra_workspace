# 团评 — 接口设计文档 v1.0

> 基于：团评系统完整需求文档集合（系统概述 + 10大业务模块 + 状态机/评分/场地/投票/关注规则）

---

## 目录

1. [通用规范](#1-通用规范)
2. [User 用户模块](#2-user-用户模块)
3. [Module 模组模块](#3-module-模组模块)
4. [GM GM模块](#4-gm-gm模块)
5. [Session 开团模块](#5-session-开团模块)
6. [Rating 评分模块](#6-rating-评分模块)
7. [Vote 投票模块](#7-vote-投票模块)
8. [Follow 关注模块](#8-follow-关注模块)
9. [Notify 通知模块](#9-notify-通知模块)
10. [Venue 场地模块](#10-venue-场地模块)
11. [RBAC 权限模块](#11-rbac-权限模块)

---

## 1. 通用规范

### 1.1 基础信息

| 项目 | 值 |
|------|-----|
| 基础URL | `/api/v1` |
| 认证方式 | JWT Bearer Token（Header: `Authorization: Bearer <token>`） |
| 内容类型 | `application/json` |
| 字符编码 | UTF-8 |

### 1.2 统一响应格式

**成功（非分页）**:
```json
{
  "code": 0,
  "message": "ok",
  "data": { ... }
}
```

**成功（分页列表）**:
```json
{
  "code": 0,
  "data": {
    "items": [...],
    "total": 150,
    "page": 1,
    "pageSize": 20
  }
}
```

**错误**:
```json
{
  "code": 40001,
  "message": "参数校验失败",
  "data": null
}
```

### 1.3 统一分页与排序参数

所有列表接口均支持以下查询参数（除非特别说明）：

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| page | int | 否 | 1 | 页码，从1开始 |
| pageSize | int | 否 | 20 | 每页条数，最大100 |
| sortBy | string | 否 | - | 排序字段 |
| sortOrder | string | 否 | desc | 排序方向：asc / desc |

### 1.4 全局错误码段分配

| 错误码范围 | 模块 |
|------------|------|
| 10001~10099 | User 用户模块 |
| 20001~20099 | Module 模组模块 |
| 30001~30099 | GM GM模块 |
| 40001~40100 | Session 开团模块 |
| 50001~50099 | Rating 评分模块 |
| 60001~60099 | Vote 投票模块 |
| 70001~70099 | Follow 关注模块 |
| 80001~80099 | Notify 通知模块 |
| 90001~90100 | Venue 场地模块 |
| 99001~99099 | RBAC 权限模块 |

### 1.5 通用错误码

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 40001 | 参数校验失败 |
| 40101 | 未登录或Token无效 |
| 40301 | 无权限访问 |
| 40401 | 资源不存在 |
| 50001 | 服务器内部错误 |

---

## 2. User 用户模块

> 对应页面：P01 登录注册、P02 个人中心、A01 后台登录、A02 用户管理

### POST /api/v1/auth/wechat-login

**接口名称**: 微信小程序登录  
**功能描述**: 通过微信code换取openid，完成登录或自动注册（小程序端）  
**使用端**: 小程序端  
**权限要求**: 无需认证  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| code | string | 是 | wx.login()获取的临时凭证 |
| userInfo.nickname | string | 否 | 昵称（首次注册时使用） |
| userInfo.avatarUrl | string | 否 | 头像URL（首次注册时使用） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 7200,
    "user": {
      "userId": "u_10001",
      "openId": "oXXXX",
      "nickname": "玩家A",
      "avatarUrl": "https://...",
      "role": "player",
      "isNewUser": false
    }
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 10001 | 微信code无效或已过期 |
| 10002 | 微信服务调用失败 |

---

### POST /api/v1/auth/web-login

**接口名称**: Web后台账号密码登录  
**功能描述**: 管理员通过账号密码登录Web管理后台  
**使用端**: Web后台  
**权限要求**: 无需认证  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | string | 是 | 登录账号 |
| password | string | 是 | 密码（MD5+盐） |
| captchaKey | string | 否 | 验证码key（连续3次失败后必填） |
| captchaCode | string | 否 | 验证码值 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 28800,
    "user": {
      "userId": "u_90001",
      "username": "admin",
      "nickname": "超级管理员",
      "role": "super_admin",
      "permissions": ["*"]
    }
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 10003 | 账号或密码错误 |
| 10004 | 账号已被禁用 |
| 10005 | 验证码错误或已过期 |

---

### POST /api/v1/auth/logout

**接口名称**: 注销登录  
**功能描述**: 使当前Token失效，清除服务端会话  
**使用端**: 两端  
**权限要求**: 已登录  

**请求参数**: 无  

**返回值**:
```json
{ "code": 0, "message": "ok" }
```

---

### GET /api/v1/user/profile

**接口名称**: 获取个人信息  
**功能描述**: 获取当前登录用户的完整资料  
**使用端**: 两端  
**权限要求**: 已登录  

**返回值**:
```json
{
  "code": 0,
  "data": {
    "userId": "u_10001",
    "openId": "oXXXX",
    "nickname": "玩家A",
    "avatarUrl": "https://...",
    "realName": "",
    "phone": "138****1234",
    "gender": 1,
    "role": "player",
    "isGmCertified": false,
    "gmInfo": null,
    "stats": {
      "sessionCount": 12,
      "moduleRated": 8,
      "followerCount": 23,
      "followingCount": 5
    },
    "createdAt": "2025-06-15T10:30:00"
  }
}
```

---

### PUT /api/v1/user/profile

**接口名称**: 更新个人信息  
**功能描述**: 修改昵称、头像、性别等基础信息  
**使用端**: 两端  
**权限要求**: 已登录  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| nickname | string | 否 | 昵称（2~20字符） |
| avatarUrl | string | 否 | 头像URL |
| gender | int | 否 | 性别：0未知 1男 2女 |
| realName | string | 否 | 真实姓名 |

**返回值**:
```json
{ "code": 0, "message": "更新成功" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 10006 | 昵称已被占用 |
| 10007 | 参数格式不合法 |

---

### GET /api/v1/user/{userId}/public

**接口名称**: 获取用户公开主页  
**功能描述**: 获取指定用户的公开信息（其他用户可查看）  
**使用端**: 小程序端  
**权限要求**: 已登录  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| userId | string | 目标用户ID |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "userId": "u_10002",
    "nickname": "GM老王",
    "avatarUrl": "https://...",
    "role": "gm",
    "isGmCertified": true,
    "stats": {
      "sessionCount": 45,
      "followerCount": 128
    },
    "recentSessions": [
      { "sessionId": "s_001", "moduleName": "诡镇疑云", "status": "completed", "completedAt": "2025-07-08T22:00:00" }
    ]
  }
}
```

---

### PUT /api/v1/user/password

**接口名称**: 修改密码  
**功能描述**: Web后台用户修改登录密码  
**使用端**: Web后台  
**权限要求**: 已登录（仅 super_admin / module_admin / cashier）  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| oldPassword | string | 是 | 原密码 |
| newPassword | string | 是 | 新密码（8~32字符，含大小写和数字） |

**返回值**:
```json
{ "code": 0, "message": "密码修改成功" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 10008 | 原密码不正确 |
| 10009 | 新密码格式不符合要求 |

---

### GET /api/v1/users

**接口名称**: 用户列表（后台管理）  
**功能描述**: 分页查询所有用户，支持按角色/状态筛选  
**使用端**: Web后台  
**权限要求**: super_admin 或 module_admin  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 搜索关键词（昵称/手机号） |
| role | string | 否 | 角色筛选：player/gm/cashier/module_admin/super_admin |
| status | int | 否 | 状态：0禁用 1正常 |
| sortBy | string | 否 | 排序字段：createdAt/lastLoginAt |
| sortOrder | string | 否 | asc/desc |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "userId": "u_10001",
        "nickname": "玩家A",
        "phone": "138****1234",
        "role": "player",
        "status": 1,
        "createdAt": "2025-06-15T10:30:00",
        "lastLoginAt": "2025-07-11T20:00:00"
      }
    ],
    "total": 520,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### PUT /api/v1/users/{userId}/status

**接口名称**: 启用/禁用用户  
**功能描述**: 管理员修改用户账户状态  
**使用端**: Web后台  
**权限要求**: super_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| userId | string | 目标用户ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| status | int | 是 | 0禁用 1启用 |

**返回值**:
```json
{ "code": 0, "message": "操作成功" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 10010 | 不能操作超级管理员账户 |
| 10011 | 不能禁用自己 |

---

### PUT /api/v1/users/{userId}/role

**接口名称**: 修改用户角色  
**功能描述**: 管理员调整用户角色（升降级GM/收银等）  
**使用端**: Web后台  
**权限要求**: super_admin |

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| userId | string | 目标用户ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| role | string | 是 | 目标角色：player/gm/cashier/module_admin |

**返回值**:
```json
{ "code": 0, "message": "角色已更新" }
```

---

## 3. Module 模组模块

> 对应页面：P03 模组首页/发现、P04 模组详情、P05 我的模组、A03 模组管理、A04 模组审核

### GET /api/v1/modules

**接口名称**: 模组列表（首页/发现）  
**功能描述**: 分页获取已上架模组，支持搜索和多维度筛选排序  
**使用端**: 小程序端  
**权限要求**: 已登录（部分公开数据可免登录查看摘要）  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 搜索关键词（标题/标签/作者） |
| categoryId | int | 否 | 分类ID |
| difficulty | int | 否 | 难度等级：1入门 2进阶 3核心 4专家 |
| playerCount | string | 否 | 人数区间：1-4/3-6/5-8/6-10 |
| duration | string | 否 | 时长区间：0-3h/3-6h/6h+ |
| tags | string | 否 | 标签筛选（逗号分隔tagId） |
| sortBy | string | 否 | 排序：latest/hottest/rating/duration |
| sortOrder | string | 否 | asc/desc |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "moduleId": "m_001",
        "title": "诡镇疑云",
        "coverImage": "https://...",
        "categoryName": "现代悬疑",
        "difficulty": 2,
        "difficultyText": "进阶",
        "minPlayers": 4,
        "maxPlayers": 7,
        "durationHours": 4,
        "authorName": "GM老王",
        "authorId": "u_10002",
        "avgRating": 4.6,
        "ratingCount": 128,
        "sessionCount": 56,
        "tags": ["推理", "恐怖", "情感"],
        "isFollowedAuthor": false
      }
    ],
    "total": 320,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### GET /api/v1/modules/{moduleId}

**接口名称**: 模组详情  
**功能描述**: 获取模组完整信息，包含评分统计、近期团等  
**使用端**: 小程序端  
**权限要求**: 已登录  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "moduleId": "m_001",
    "title": "诡镇疑云",
    "subtitle": "迷雾笼罩的小镇，谁在说谎？",
    "coverImage": "https://...",
    "images": ["https://...", "https://..."],
    "categoryId": 3,
    "categoryName": "现代悬疑",
    "difficulty": 2,
    "difficultyText": "进阶",
    "minPlayers": 4,
    "maxPlayers": 7,
    "recommendedPlayers": "5-6人",
    "durationHours": 4,
    "description": "详细背景故事介绍...",
    "rulesSummary": "规则概述...",
    "authorId": "u_10002",
    "authorName": "GM老王",
    "authorAvatar": "https://...",
    "isGmCertified": true,
    "isFollowedAuthor": false,
    "tags": ["推理", "恐怖", "情感"],
    "status": "published",
    "createdAt": "2025-05-20T14:00:00",
    "publishedAt": "2025-05-25T10:00:00",
    "ratingStats": {
      "overall": 4.6,
      "dimensions": {
        "story": 4.8,
        "gameplay": 4.5,
        "atmosphere": 4.7,
        "value": 4.4
      },
      "totalCount": 128,
      "distribution": { "5": 68, "4": 42, "3": 15, "2": 2, "1": 1 }
    },
    "myRating": null,
    "recentSessions": [
      {
        "sessionId": "s_101",
        "gmName": "GM老王",
        "scheduledTime": "2025-07-15T19:00:00",
        "currentPlayers": 5,
        "maxPlayers": 6,
        "venueName": "包间A",
        "status": "recruiting"
      }
    ]
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20001 | 模组不存在 |
| 20002 | 模组未发布（非作者和管理员不可见） |

---

### POST /api/v1/modules

**接口名称**: 创建模组（草稿）  
**功能描述**: GM创建新模组，初始状态为draft  
**使用端**: 小程序端 / Web后台  
**权限要求**: gm 及以上角色  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| title | string | 是 | 模组标题（2~50字符） |
| subtitle | string | 否 | 副标题（最大100字符） |
| coverImage | string | 是 | 封面图URL |
| images | string[] | 否 | 详情图片列表（最多9张） |
| categoryId | int | 是 | 分类ID |
| difficulty | int | 是 | 难度：1~4 |
| minPlayers | int | 是 | 最少推荐人数（≥1） |
| maxPlayers | int | 是 | 最多推荐人数（≥minPlayers，≤30） |
| durationHours | decimal | 是 | 预计时长（小时，0.5精度） |
| description | string | 是 | 详细描述（最大5000字符） |
| rulesSummary | string | 否 | 规则概述（最大2000字符） |
| tags | int[] | 否 | 标签ID列表（最多5个） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "moduleId": "m_new001",
    "status": "draft",
    "message": "模组创建成功，请完善内容后提交审核"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20003 | 标题重复 |
| 20004 | 人数范围不合法 |
| 20005 | 标签数量超限 |

---

### PUT /api/v1/modules/{moduleId}

**接口名称**: 编辑模组  
**功能描述**: 编辑模组基本信息（仅draft/rejected状态可编辑全部字段；published状态仅可编辑部分字段）  
**使用端**: 小程序端 / Web后台  
**权限要求**: 模组作者 或 module_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**请求参数**: 同POST /api/v1/modules（均为可选，仅传需修改的字段）

**返回值**:
```json
{ "code": 0, "message": "模组更新成功" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20006 | 当前状态不允许编辑 |
| 20007 | 无权编辑此模组 |

---

### POST /api/v1/modules/{moduleId}/submit-review

**接口名称**: 提交审核  
**功能描述**: 将draft状态的模组提交至待审核队列  
**使用端**: 小程序端 / Web后台  
**权限要求**: 模组作者  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**返回值**:
```json
{ "code": 0, "message": "已提交审核，预计1~3个工作日完成" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20008 | 仅草稿状态可提交审核 |
| 20009 | 模组信息不完整，请补充必填项 |

---

### GET /api/v1/modules/mine

**接口名称**: 我的模组列表  
**功能描述**: 获取当前用户创建的所有模组（全状态）  
**使用端**: 小程序端  
**权限要求**: gm 及以上角色  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 状态筛选：draft/pending_review/published/rejected/offline |
| keyword | string | 否 | 搜索关键词 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "moduleId": "m_001",
        "title": "诡镇疑云",
        "coverImage": "https://...",
        "status": "published",
        "statusText": "已发布",
        "avgRating": 4.6,
        "sessionCount": 56,
        "submittedAt": "2025-05-24T09:00:00",
        "reviewedAt": "2025-05-25T10:00:00",
        "rejectReason": null
      }
    ],
    "total": 8,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### POST /api/v1/modules/{moduleId}/offline

**接口名称**: 下架模组  
**功能描述**: 将已发布模组下架（不影响已有团和评价）  
**使用端**: 小程序端 / Web后台  
**权限要求**: 模组作者 或 module_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| reason | string | 否 | 下架原因（管理员操作时必填） |

**返回值**:
```json
{ "code": 0, "message": "模组已下架" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20010 | 仅已发布状态可下架 |
| 20011 | 存在进行中的团，建议先处理 |

---

### POST /api/v1/modules/{moduleId}/republish

**接口名称**: 重新上架  
**功能描述**: 将offline状态模组重新进入审核流程  
**使用端**: 小程序端 / Web后台  
**权限要求**: 模组作者 或 module_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**返回值**:
```json
{ "code": 0, "message": "已重新提交审核" }
```

---

### DELETE /api/v1/modules/{moduleId}

**接口名称**: 删除模组  
**功能描述**: 删除草稿/被拒/下架且无关联团的模组  
**使用端**: 小程序端 / Web后台  
**权限要求**: 模组作者 或 super_admin  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**返回值**:
```json
{ "code": 0, "message": "模组已删除" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20012 | 存在关联的团记录，不可删除 |
| 20013 | 仅草稿/拒绝/下架状态可删除 |

---

### GET /api/v1/modules/categories

**接口名称**: 模组分类列表  
**功能描述**: 获取所有模组分类及各分类下的模组数量  
**使用端**: 两端  
**权限要求**: 已登录（分类列表可公开缓存）  

**返回值**:
```json
{
  "code": 0,
  "data": [
    { "categoryId": 1, "name": "古典奇幻", "icon": "🗡️", "moduleCount": 45 },
    { "categoryId": 2, "name": "现代悬疑", "icon": "🔍", "moduleCount": 38 },
    { "categoryId": 3, "name": "科幻未来", "icon": "🚀", "moduleCount": 27 },
    { "categoryId": 4, "name": "恐怖惊悚", "icon": "👻", "moduleCount": 32 },
    { "categoryId": 5, "name": "恋爱日常", "icon": "💕", "moduleCount": 18 }
  ]
}
```

---

### GET /api/v1/modules/tags

**接口名称**: 模组标签列表  
**功能描述**: 获取所有可用标签（用于筛选和创建时选择）  
**使用端**: 两端  
**权限要求**: 已登录  

**返回值**:
```json
{
  "code": 0,
  "data": [
    { "tagId": 1, "name": "推理", "usageCount": 156 },
    { "tagId": 2, "name": "战斗", "usageCount": 98 },
    { "tagId": 3, "name": "情感", "usageCount": 87 },
    { "tagId": 4, "name": "恐怖", "usageCount": 76 }
  ]
}
```

---

### GET /api/v1/admin/modules/pending-review

**接口名称**: 待审核模组列表（后台）  
**功能描述**: 获取所有pending_review状态的模组供管理员审核  
**使用端**: Web后台  
**权限要求**: module_admin 或 super_admin  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 搜索关键词 |
| submitTimeStart | datetime | 否 | 提交时间起始 |
| submitTimeEnd | datetime | 否 | 提交时间截止 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "moduleId": "m_050",
        "title": "星际迷航：暗影",
        "authorName": "新人GM",
        "coverImage": "https://...",
        "categoryId": 3,
        "categoryName": "科幻未来",
        "submittedAt": "2025-07-10T15:30:00",
        "previewDescription": "前200字预览..."
      }
    ],
    "total": 5,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### POST /api/v1/admin/modules/{moduleId}/review

**接口名称**: 审核模组  
**功能描述**: 管理员审核模组——通过或驳回  
**使用端**: Web后台  
**权限要求**: module_admin 或 super_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| action | string | 是 | approve / reject |
| reason | string | 否 | 驳回原因（reject时必填，最大500字符） |

**返回值**:
```json
{ "code": 0, "message": "审核完成" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 20014 | 模组不在待审核状态 |
| 20015 | 驳回原因不能为空 |

---

### GET /api/v1/admin/modules

**接口名称**: 全部模组管理（后台）  
**功能描述**: 管理员查看所有模组，支持全状态筛选  
**使用端**: Web后台  
**权限要求**: module_admin 或 super_admin  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 关键词 |
| status | string | 否 | 状态筛选 |
| authorId | string | 否 | 作者ID |
| categoryId | int | 否 | 分类 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 同GET /api/v1/modules/mine结构，增加 `authorName`, `reviewerName`, `reviewedAt` 等管理字段。

---

## 4. GM GM模块

> 对应页面：P06 GM主页、P07 GM信息编辑、A05 GM管理

### GET /api/v1/gm/profile

**接口名称**: 获取我的GM资料  
**功能描述**: 获取当前用户的GM身份信息和认证状态  
**使用端**: 小程序端  
**权限要求**: gm 角色  

**返回值**:
```json
{
  "code": 0,
  "data": {
    "userId": "u_10002",
    "nickname": "GM老王",
    "avatarUrl": "https://...",
    "gmProfile": {
      "bio": "8年DM经验，擅长克苏鲁与现代悬疑跑团",
      "experienceYears": 8,
      "goodAtCategories": ["现代悬疑", "恐怖惊悚"],
      "goodAtTags": ["推理", "沉浸"],
      "certificationStatus": "approved",
      "certificationAppliedAt": "2025-04-01T10:00:00",
      "certificationReviewedAt": "2025-04-02T14:00:00",
      "idCardName": "王**",
      "idCardNumber": "**************1234",
      "idCardImageFront": "https://...",
      "idCardImageBack": "https://..."
    },
    "stats": {
      "totalSessions": 45,
      "completedSessions": 42,
      "avgGmRating": 4.7,
      "followerCount": 128,
      "moduleCount": 6
    },
    "recentSessions": [
      { "sessionId": "s_101", "moduleName": "诡镇疑云", "status": "completed", "completedAt": "2025-07-08T22:00:00" }
    ]
  }
}
```

---

### PUT /api/v1/gm/profile

**接口名称**: 编辑GM资料  
**功能描述**: 修改GM个人简介、擅长领域等展示信息  
**使用端**: 小程序端  
**权限要求**: gm 角色  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| bio | string | 否 | 个人简介（最大500字符） |
| experienceYears | int | 否 | 经验年数（0~50） |
| goodAtCategories | string[] | 否 | 擅长分类名称列表 |
| goodAtTags | string[] | 否 | 擅长标签列表 |

**返回值**:
```json
{ "code": 0, "message": "GM资料已更新" }
```

---

### POST /api/v1/gm/certification/apply

**接口名称**: 申请GM认证  
**功能描述**: 提交实名认证材料申请GM认证标识  
**使用端**: 小程序端  
**权限要求**: gm 角色，当前未认证  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| realName | string | 是 | 真实姓名 |
| idCardNumber | string | 是 | 身份证号码 |
| idCardImageFront | string | 是 | 身份证正面照URL |
| idCardImageBack | string | 是 | 身份证背面照URL |

**返回值**:
```json
{ "code": 0, "message": "认证申请已提交，预计1~3个工作日" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 30001 | 已有进行中的认证申请 |
| 30002 | 身份证号格式不正确 |
| 30003 | 认证材料不完整 |

---

### GET /api/v1/gm/{gmId}/public

**接口名称**: GM公开主页  
**功能描述**: 查看任意GM的公开主页信息  
**使用端**: 小程序端  
**权限要求**: 已登录  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| gmId | string | GM用户ID |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "userId": "u_10002",
    "nickname": "GM老王",
    "avatarUrl": "https://...",
    "bio": "8年DM经验...",
    "experienceYears": 8,
    "isGmCertified": true,
    "isFollowed": false,
    "goodAtCategories": ["现代悬疑", "恐怖惊悚"],
    "goodAtTags": ["推理", "沉浸"],
    "stats": {
      "totalSessions": 45,
      "completedSessions": 42,
      "avgGmRating": 4.7,
      "followerCount": 128,
      "moduleCount": 6
    },
    "ratingDimensions": {
      "narration": 4.8,
      "improvisation": 4.6,
      "fairness": 4.9,
      "rhythm": 4.5,
      "characterPortrayal": 4.7,
      "atmosphere": 4.8
    },
    "modules": [
      { "moduleId": "m_001", "title": "诡镇疑云", "avgRating": 4.6, "sessionCount": 30 }
    ],
    "upcomingSessions": [
      { "sessionId": "s_201", "moduleName": "诡镇疑云", "scheduledTime": "2025-07-15T19:00:00", "currentPlayers": 5, "maxPlayers": 6, "status": "recruiting" }
    ]
  }
}
```

---

### GET /api/v1/admin/gms

**接口名称**: GM列表管理（后台）  
**功能描述**: 管理员查看所有GM，支持认证状态筛选  
**使用端**: Web后台  
**权限要求**: module_admin 或 super_admin  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 搜索关键词 |
| certificationStatus | string | 否 | 认证状态：pending/approved/rejected/none |
| isInternal | bool | 否 | 是否内部GM（免场地费） |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "userId": "u_10002",
        "nickname": "GM老王",
        "phone": "139****5678",
        "certificationStatus": "approved",
        "isInternal": false,
        "sessionCount": 45,
        "avgGmRating": 4.7,
        "followerCount": 128,
        "createdAt": "2025-03-01T10:00:00"
      }
    ],
    "total": 35,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### POST /api/v1/admin/gms/{gmId}/certification/review

**接口名称**: 审核GM认证  
**功能描述**: 管理员审批GM实名认证申请  
**使用端**: Web后台  
**权限要求**: super_admin 或 module_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| gmId | string | GM用户ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| action | string | 是 | approve / reject |
| reason | string | 否 | 驳回原因 |

**返回值**:
```json
{ "code": 0, "message": "认证审核完成" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 30004 | 该用户无待审核的认证申请 |

---

### PUT /api/v1/admin/gms/{gmId}/internal-flag

**接口名称**: 设置内部GM标识  
**功能描述**: 将GM标记为内部GM（免场地费）或取消  
**使用端**: Web后台  
**权限要求**: super_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| gmId | string | GM用户ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| isInternal | bool | 是 | true=内部GM(免场地费) false=普通GM |

**返回值**:
```json
{ "code": 0, "message": "内部GM设置已更新" }
```

---

## 5. Session 开团模块

> 对应页面：P08 开团/我的团、P09 团详情、P10 报名列表、A06 团管理、A07 报名管理、A08 场地预约

### Session状态机说明

```
                    ┌─────────────┐
                    │   draft     │ (草稿)
                    └──────┬──────┘
                           │ 提交开团(需付场地费)
                           ▼
                    ┌─────────────┐
              ┌────│ pending_     │
              │    │ payment      │ (待支付)
              │    └──────┬──────┘
              │           │ 支付成功
              │           ▼
              │    ┌─────────────┐
         支付超时│    │ recruiting  │ ◄────────────┐
              │    │             │ (招募中)       │
              │    └──┬──────┬───┘               │
              │       │      │ 人数满             │
              │       │      ▼                   │
              │       │ ┌─────────┐              │
              │       │ │  full   │              │
              │       │ │         │(满员)        │
              │       │ └────┬────┘              │
              │       │      │ GM确认            │
              │       │      ▼                   │
              │       │ ┌─────────────┐          │
              │       │ │ confirmed   │◄─────────┘
              │       │ │             │(已确认)   │ 有人取消→回recruiting
              │       │ └──────┬──────┘
              │              │ 到时间开始
              │              ▼
              │       ┌──────────────┐
              │       │ in_progress  │
              │       │              │(进行中)
              │       └──────┬───────┘
              │              │ 正常结束
              │              ▼
              │       ┌──────────────┐
              └──────►│  completed   │(终态)
                      │              │
                      └──────────────┘

终态分支：
  cancelled  - 取消
  failed     - 失败（如GM缺席）
  rescheduled- 改期（生成新团）
过渡态：
  expired    - 过期（支付超时/招募超时自动转入）
```

---

### POST /api/v1/sessions

**接口名称**: 创建开团（草稿）  
**功能描述**: GM创建新团，初始状态为draft  
**使用端**: 小程序端  
**权限要求**: gm 角色  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| moduleId | string | 是 | 模组ID（必须为published状态） |
| title | string | 否 | 团标题（默认取模组名） |
| description | string | 否 | 补充说明（最大500字符） |
| scheduledTime | datetime | 是 | 计划开始时间 |
| estimatedEndtime | datetime | 否 | 预计结束时间 |
| minPlayers | int | 是 | 开团最少人数（≥1，≤模组minPlayers时不限制，但不得大于模组max_players） |
| maxPlayers | int | 是 | 开团最多人数（≥minPlayers，≤模组max_players，≤room.capacity） |
| venueRoomId | int | 否 | 包间ID（选填，不填后续再选） |
| pricePerPlayer | decimal | 否 | 人均费用（0表示免费） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_new001",
    "status": "draft",
    "message": "团已创建，请确认信息后提交"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 模组不存在或未发布 |
| 40002 | 人数范围不合法（min > max / max > 模组max_players） |
| 40003 | 计划时间不能是过去时间 |
| 40004 | 包间容量不足（maxPlayers > room.capacity） |

---

### PUT /api/v1/sessions/{sessionId}

**接口名称**: 编辑开团信息  
**功能描述**: 编辑draft状态的开团信息  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**: 同POST /api/v1/sessions（均为可选）

**业务规则**:
- 编辑时 `minPlayers` 不得大于当前已报名人数
- 若已有人报名，`maxPlayers` 不得小于当前已报名人数

**返回值**:
```json
{ "code": 0, "message": "团信息已更新" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40005 | 仅草稿状态可编辑 |
| 40006 | minPlayers不得大于当前已报名人数 |
| 40007 | 不是该团的GM |

---

### POST /api/v1/sessions/{sessionId}/submit

**接口名称**: 提交开团（进入待支付）  
**功能描述**: 将draft团提交，若选择了包间则进入待支付状态  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| venueRoomId | int | 是 | 包间ID（提交时必须选定） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_new001",
    "status": "pending_payment",
    "paymentInfo": {
      "orderId": "pay_001",
      "amount": 88.00,
      "currency": "CNY",
      "paymentMethod": "wechat_mini",
      "expireMinutes": 30
    }
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40008 | 包间在该时段已被占用 |
| 40009 | 非内部GM必须支付场地费 |
| 40010 | 团信息不完整 |

---

### POST /api/v1/sessions/{sessionId}/confirm-start

**接口名称**: GM确认开团（full → confirmed）  
**功能描述**: 当团满员后，GM手动确认准备开始  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID（状态须为full） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_001",
    "status": "confirmed",
    "message": "团已确认，请准时开始"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40011 | 当前状态不允许此操作 |
| 40012 | 人数不足无法确认 |

---

### POST /api/v1/sessions/{sessionId}/start

**接口名称**: 开始团（confirmed → in_progress）  
**功能描述**: GM在预定时间点击开始，团进入进行中状态  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID（状态须为confirmed） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_001",
    "status": "in_progress",
    "actualStartTime": "2025-07-15T19:05:00",
    "message": "团已正式开始！祝大家玩得开心"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40013 | 未到预定开始时间（允许提前15分钟内开始） |
| 40014 | 当前状态不允许开始 |

---

### POST /api/v1/sessions/{sessionId}/complete

**接口名称**: 结束团（in_progress → completed）  
**功能描述**: GM结束团，进入已完成终态，触发评分入口开放  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID（状态须为in_progress） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_001",
    "status": "completed",
    "actualEndTime": "2025-07-15T23:30:00",
    "durationMinutes": 265,
    "message": "团已结束，感谢参与！快去给模组和GM评分吧~"
  }
}
```

---

### POST /api/v1/sessions/{sessionId}/cancel

**接口名称**: 取消团  
**功能描述**: GM取消团（draft/recruiting/full/confirmed状态），转入cancelled终态  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM) 或 super_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| reason | string | 是 | 取消原因（最大200字符） |

**业务规则**:
- 距 original_scheduled_time < 48小时不可取消
- 自动触发退款流程（若有已支付费用）
- 自动释放包间锁定
- 向所有已报名玩家发送通知

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_001",
    "status": "cancelled",
    "refundTriggered": true,
    "notifiedCount": 5
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40015 | 距开始不足48小时，不可取消 |
| 40016 | 当前状态不允许取消 |
| 40017 | 只有GM可以取消自己的团 |

---

### POST /api/v1/sessions/{sessionId}/reschedule

**接口名称**: 改期  
**功能描述**: 将团改为新时间，原团变为rescheduled终态，生成新团继承报名信息  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| newScheduledTime | datetime | 是 | 新的计划开始时间 |
| reason | string | 是 | 改期原因 |
| keepRegistrations | bool | 否 | 是否保留原报名人员（默认true） |

**业务规则**:
- 距 original_scheduled_time < 48小时不可改期
- 每个Session限改期1次
- 需重新检查包间冲突
- 原团状态→rescheduled，新建一团继承信息

**返回值**:
```json
{
  "code": 0,
  "data": {
    "oldSessionId": "s_001",
    "oldSessionStatus": "rescheduled",
    "newSessionId": "s_002",
    "newScheduledTime": "2025-07-22T19:00:00",
    "registrationsKept": 5,
    "message": "改期成功，已通知所有已报名玩家"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40018 | 距开始不足48小时，不可改期 |
| 40019 | 该团已经改期过，不可再次改期 |
| 40020 | 新时间段包间已被占用 |
| 40021 | 当前状态不允许改期 |

---

### GET /api/v1/sessions/{sessionId}

**接口名称**: 团详情  
**功能描述**: 获取团的完整信息，根据角色返回不同视角的数据  
**使用端**: 小程序端 / Web后台  
**权限要求**: 已登录（部分信息对报名者可见）  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_001",
    "title": "诡镇疑云 - 0715场",
    "status": "recruiting",
    "statusText": "招募中",
    "moduleId": "m_001",
    "moduleName": "诡镇疑云",
    "moduleCoverImage": "https://...",
    "gmId": "u_10002",
    "gmName": "GM老王",
    "gmAvatar": "https://...",
    "isGmCertified": true,
    "description": "新手友好，带好零食~",
    "originalScheduledTime": "2025-07-15T19:00:00",
    "scheduledTime": "2025-07-15T19:00:00",
    "estimatedEndTime": "2025-07-15T23:00:00",
    "minPlayers": 4,
    "maxPlayers": 6,
    "currentPlayers": 5,
    "pricePerPlayer": 30.00,
    "venueRoomId": 1,
    "venueRoomName": "包间A",
    "venueAddress": "北京市朝阳区xxx",
    "registrationList": [
      {
        "registrationId": "r_001",
        "userId": "u_10003",
        "nickname": "玩家B",
        "avatarUrl": "https://...",
        "status": "confirmed",
        "registeredAt": "2025-07-10T14:00:00",
        "confirmedAt": "2025-07-10T16:00:00",
        "isSelf": false,
        "paymentStatus": "paid"
      }
    ],
    "myRegistration": {
      "registrationId": "r_002",
      "status": "approved",
      "registeredAt": "2025-07-11T10:00:00",
      "canCancel": true,
      "canRate": false
    },
    "isFollowedGm": false,
    "createdAt": "2025-07-08T20:00:00",
    "rescheduleCount": 0,
    "paymentInfo": {
      "orderId": "pay_001",
      "amount": 88.00,
      "status": "paid",
      "paidAt": "2025-07-08T20:05:00"
    }
  }
}
```

---

### GET /api/v1/sessions

**接口名称**: 团列表（发现/浏览）  
**功能描述**: 分页获取招募中的团，支持多维度筛选  
**使用端**: 小程序端  
**权限要求**: 已登录  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 搜索关键词（团名/模组名/GM名） |
| moduleId | string | 否 | 按模组筛选 |
| gmId | string | 否 | 按GM筛选 |
| status | string | 否 | 状态筛选（默认recruiting,full,confirmed） |
| dateFrom | date | 否 | 开始日期起 |
| dateTo | date | 否 | 开始日期止 |
| hasVacancy | bool | 否 | 是否仅显示有空位的团 |
| sortBy | string | 否 | 排序：scheduledTime/latest/hot |
| sortOrder | string | 否 | asc/desc |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "sessionId": "s_001",
        "title": "诡镇疑云 - 0715场",
        "moduleId": "m_001",
        "moduleName": "诡镇疑云",
        "moduleCoverImage": "https://...",
        "gmId": "u_10002",
        "gmName": "GM老王",
        "gmAvatar": "https://...",
        "isGmCertified": true,
        "scheduledTime": "2025-07-15T19:00:00",
        "minPlayers": 4,
        "maxPlayers": 6,
        "currentPlayers": 5,
        "pricePerPlayer": 30.00,
        "venueRoomName": "包间A",
        "status": "recruiting"
      }
    ],
    "total": 89,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### GET /api/v1/sessions/mine/gm

**接口名称**: 我开的团（GM视角）  
**功能描述**: 获取当前用户作为GM创建的所有团（全状态）  
**使用端**: 小程序端  
**权限要求**: gm 角色  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 状态筛选 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "sessionId": "s_001",
        "title": "诡镇疑云 - 0715场",
        "moduleName": "诡镇疑云",
        "status": "recruiting",
        "scheduledTime": "2025-07-15T19:00:00",
        "currentPlayers": 5,
        "maxPlayers": 6,
        "venueRoomName": "包间A",
        "createdAt": "2025-07-08T20:00:00"
      }
    ],
    "total": 12,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### GET /api/v1/sessions/mine/player

**接口名称**: 我参与的团（玩家视角）  
**功能描述**: 获取当前用户报名参与过的所有团  
**使用端**: 小程序端  
**权限要求**: 已登录  

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 状态筛选 |
| upcomingOnly | bool | 否 | 是否仅显示即将开始的团 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 同上结构，增加 `myRegistrationStatus` 字段。

---

### POST /api/v1/sessions/{sessionId}/register

**接口名称**: 报名参加团  
**功能描述**: 玩家报名加入一个招募中的团  
**使用端**: 小程序端  
**权限要求**: 已登录（player 或 gm 角色）  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID（状态须为recruiting） |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| remark | string | 否 | 备注信息（最大100字符） |

**业务规则**:
- 团状态须为 recruiting
- 当前人数 < maxPlayers
- 不能重复报名
- GM不能报自己的团

**返回值**:
```json
{
  "code": 0,
  "data": {
    "registrationId": "r_new001",
    "status": "pending",
    "message": "报名成功，等待GM确认",
    "currentPlayers": 6,
    "isFull": true
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40022 | 团不在招募中 |
| 40023 | 团已满员 |
| 40024 | 已报名过该团 |
| 40025 | 不能报名自己开的团 |

---

### POST /api/v1/sessions/{sessionId}/register/proxy

**接口名称**: 代报名  
**功能描述**: GM/管理员代其他玩家报名  
**使用端**: 小程序端 / Web后台  
**权限要求**: gm 角色（仅限自己的团）或 super_admin  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| targetUserId | string | 是 | 被代报名的用户ID |
| remark | string | 否 | 备注 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "registrationId": "r_proxy001",
    "message": "代报名成功"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40026 | 目标用户不存在 |
| 40027 | 目标用户已报名该团 |

---

### POST /api/v1/sessions/{sessionId}/registrations/{registrationId}/confirm

**接口名称**: 确认报名  
**功能描述**: GM确认或拒绝玩家的报名申请  
**使用端**: 小程序端  
**权限要求**: 团的创建者(GM)  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |
| registrationId | string | 报名记录ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| action | string | 是 | approve / reject |
| reason | string | 否 | 拒绝原因 |

**返回值**:
```json
{ "code": 0, "message": "已确认报名" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40028 | 报名记录不存在或不属于此团 |
| 40029 | 该报名已处理过 |

---

### POST /api/v1/sessions/{sessionId}/registrations/{registrationId}/cancel

**接口名称**: 取消报名  
**功能描述**: 玩家取消自己的报名，或GM移除已确认的玩家  
**使用端**: 小程序端  
**权限要求**: 报名人本人 或 团的GM  

**路径参数**:
| 参数名 | 类型 | 说明 |
|--------|------|------|
| sessionId | string | 团ID |
| registrationId | string | 报名记录ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| reason | string | 否 | 取消原因 |

**业务规则**:
- 仅 recruiting/full/confirmed 状态可取消
- 取消后团从full→recruiting（若之前满员）
- 向GM和其他玩家发送通知

**返回值**:
```json
{ "code": 0, "message": "报名已取消" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40030 | 当前状态不允许取消报名 |
| 40031 | 无权操作此报名 |

---

### GET /api/v1/sessions/{sessionId}/registrations

**接口名称**: 报名列表  
**功能描述**: 获取团的全部报名记录（GM视角含详细信息）  
**使用端**: 小程序端 / Web后台  
**权限要求**: 团的GM、被报名玩家本人、或管理员  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 报名状态筛选：pending/approved/rejected/cancelled |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "registrationId": "r_001",
        "userId": "u_10003",
        "nickname": "玩家B",
        "avatarUrl": "https://...",
        "status": "approved",
        "registeredAt": "2025-07-10T14:00:00",
        "confirmedAt": "2025-07-10T16:00:00",
        "remark": "第一次玩这个模组",
        "paymentStatus": "paid",
        "isSelf": false
      }
    ],
    "total": 5,
    "sessionInfo": {
      "maxPlayers": 6,
      "currentPlayers": 5,
      "status": "recruiting"
    }
  }
}
```

---

### GET /api/v1/admin/sessions

**接口名称**: 全部团管理（后台）  
**功能描述**: 管理员查看所有团，支持多条件筛选  
**使用端**: Web后台  
**权限要求**: module_admin 或 super_admin  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| keyword | string | 否 | 关键词 |
| status | string | 否 | 状态筛选 |
| gmId | string | 否 | GM筛选 |
| moduleId | string | 否 | 模组筛选 |
| dateFrom | date | 否 | 开始日期起 |
| dateTo | date | 否 | 开始日期止 |
| venueRoomId | int | 否 | 包间筛选 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 同GET /api/v1/sessions结构，增加管理字段 `paymentAmount`, `refundStatus`, `adminNotes`。

---

### POST /api/v1/admin/sessions/{sessionId}/force-status

**接口名称**: 强制修改团状态（后台）  
**功能描述**: 管理员强制变更团状态（应急操作）  
**使用端**: Web后台  
**权限要求**: super_admin  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| sessionId | string | 团ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| targetStatus | string | 是 | 目标状态 |
| reason | string | 是 | 操作原因 |

**返回值**:
```json
{ "code": 0, "message": "状态已强制修改" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40032 | 非法状态转移 |

---

## 6. Rating 评分模块

> 对应页面：P11 评分页面、P12 我的评分、A09 评分管理

### 评分机制核心规则回顾

| 维度 | 权重说明 |
|------|----------|
| **体验层级权重** | 已体验3.0 : 别处体验1.5 : 未体验1.0 |
| **评价关系矩阵** | 已参与者→模组+GM+玩家；别处体验→仅模组；未体验→仅模组 |
| **GM评分限制** | "别处体验"用户不可评GM |
| **修改规则** | 已体验72h内可多次修改；别处/未体验24h内仅1次 |
| **删除规则** | 超出修改窗口后不可删除（除非管理员操作） |

---

### POST /api/v1/ratings/module

**接口名称**: 评价模组  
**功能描述**: 对模组进行4维度评分（故事性/游戏性/氛围感/性价比）  
**使用端**: 小程序端  
**权限要求**: 已登录  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| moduleId | string | 是 | 模组ID |
| sessionId | string | 否 | 关联团ID（有则标记为"已体验"） |
| story | decimal(1) | 是 | 故事性评分 1~5（0.1精度） |
| gameplay | decimal(1) | 是 | 游戏性评分 1~5 |
| atmosphere | decimal(1) | 是 | 氛围感评分 1~5 |
| value | decimal(1) | 是 | 性价比评分 1~5 |
| content | string | 否 | 文字评价（最大500字符） |
| isAnonymous | bool | 否 | 是否匿名（默认false） |

**业务规则**:
- 同一用户对同一模组，按体验层级分别计分
- 若传sessionId且用户参与了该团 → 已体验(3.0权重)
- 若不传sessionId但用户在其他团玩过该模组 → 别处体验(1.5权重)
- 都没有 → 未体验(1.0权重)

**返回值**:
```json
{
  "code": 0,
  "data": {
    "ratingId": "rt_m001",
    "experienceType": "participated",
    "weight": 3.0,
    "message": "评价提交成功"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 50001 | 模组不存在 |
| 50002 | 评分超出范围（须1~5） |
| 50003 | 关联的团未完成（不能对进行中的团评分） |
| 50004 | 已超过修改窗口期（别处/未体验24h后不可修改） |

---

### PUT /api/v1/ratings/module/{ratingId}

**接口名称**: 修改模组评分  
**功能描述**: 在允许的时间窗口内修改已有的模组评分  
**使用端**: 小程序端  
**权限要求**: 评价者本人  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| ratingId | string | 评分记录ID |

**请求参数**: 同POST /api/v1/ratings/module（均为可选）

**业务规则**:
- 已体验：72小时内可多次修改
- 别处/未体验：24小时内仅可修改1次

**返回值**:
```json
{ "code": 0, "message": "评分已更新" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 50005 | 已超过修改窗口期 |
| 50006 | 别处/未体验评分修改次数已用尽 |
| 50007 | 无权修改此评分 |

---

### DELETE /api/v1/ratings/module/{ratingId}

**接口名称**: 删除模组评分  
**功能描述**: 删除自己的模组评分（仅限窗口期内或管理员操作）  
**使用端**: 小程序端 / Web后台  
**权限要求**: 评价者本人（窗口期内）或 super_admin  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| ratingId | string | 评分记录ID |

**返回值**:
```json
{ "code": 0, "message": "评分已删除" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 50008 | 窗口期已过，不可自行删除 |

---

### POST /api/v1/ratings/gm

**接口名称**: 评价GM  
**功能描述**: 对GM进行6维度评分（叙述能力/即兴发挥/公平性/节奏把控/角色塑造/氛围营造）  
**使用端**: 小程序端  
**权限要求**: 已登录，且对该GM所在团有"已体验"记录  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| gmId | string | 是 | 被评GM用户ID |
| sessionId | string | 是 | 关联团ID（必须是自己参与的已完成团） |
| narration | decimal(1) | 是 | 叙述能力 1~5 |
| improvisation | decimal(1) | 是 | 即兴发挥 1~5 |
| fairness | decimal(1) | 是 | 公平性 1~5 |
| rhythm | decimal(1) | 是 | 节奏把控 1~5 |
| characterPortrayal | decimal(1) | 是 | 角色塑造 1~5 |
| atmosphere | decimal(1) | 是 | 氛围营造 1~5 |
| content | string | 否 | 文字评价 |
| isAnonymous | bool | 否 | 是否匿名 |

**业务规则**:
- 仅"已体验"用户可评GM（别处体验和未体验不可）
- 每人每团每GM仅评1次

**返回值**:
```json
{
  "code": 0,
  "data": {
    "ratingId": "rt_g001",
    "message": "GM评价提交成功"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 50009 | 无权评价该GM（非已体验关系） |
| 50010 | 该团已评价过此GM |
| 50011 | 团尚未完成 |

---

### PUT /api/v1/ratings/gm/{ratingId}

**接口名称**: 修改GM评分  
**功能描述**: 72小时内修改GM评分  
**使用端**: 小程序端  
**权限要求**: 评价者本人  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| ratingId | string | 评分记录ID |

**请求参数**: 同POST /api/v1/ratings/gm（均为可选）

**返回值**:
```json
{ "code": 0, "message": "GM评分已更新" }
```

---

### POST /api/v1/ratings/player

**接口名称**: 评价玩家（互评）  
**功能描述**: 对同团参与者进行5维度评分（角色扮演/配合度/规则遵守/情绪贡献/准时守约）  
**使用端**: 小程序端  
**权限要求**: 已登录，且与目标玩家在同一已完成团中  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| targetUserId | string | 是 | 被评玩家ID |
| sessionId | string | 是 | 关联团ID（必须两人都参与且已完成） |
| roleplay | decimal(1) | 是 | 角色扮演 1~5 |
| cooperation | decimal(1) | 是 | 配合度 1~5 |
| ruleFollowing | decimal(1) | 是 | 规则遵守 1~5 |
| moodContribution | decimal(1) | 是 | 情绪贡献 1~5 |
| punctuality | decimal(1) | 是 | 准时守约 1~5 |
| content | string | 否 | 文字评价 |
| isAnonymous | bool | 否 | 是否匿名 |

**业务规则**:
- 仅"已体验"用户可互评
- 不能评自己
- 每人每团每玩家仅评1次

**返回值**:
```json
{
  "code": 0,
  "data": {
    "ratingId": "rt_p001",
    "message": "玩家评价提交成功"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 50012 | 不能评价自己 |
| 50013 | 与目标玩家无共同参与的已完成团 |
| 50014 | 已评价过该玩家在此团的表现 |

---

### PUT /api/v1/ratings/player/{ratingId}

**接口名称**: 修改玩家评分  
**功能描述**: 72小时内修改玩家互评  
**使用端**: 小程序端  
**权限要求**: 评价者本人  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| ratingId | string | 评分记录ID |

**请求参数**: 同POST /api/v1/ratings/player（均为可选）

**返回值**:
```json
{ "code": 0, "message": "玩家评分已更新" }
```

---

### GET /api/v1/ratings/module/{moduleId}

**接口名称**: 模组评分详情  
**功能描述**: 获取模组的完整评分统计和评分列表  
**使用端**: 两端  
**权限要求**: 已登录  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| moduleId | string | 模组ID |

**查询参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| experienceType | string | 否 | 按体验类型筛选：participated/elsewhere/unexperienced |
| sortBy | string | 否 | 排序：latest/highest/lowest |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "summary": {
      "overall": 4.6,
      "totalCount": 128,
      "weightedScore": 4.52,
      "dimensions": {
        "story": { "avg": 4.8, "count": 128 },
        "gameplay": { "avg": 4.5, "count": 125 },
        "atmosphere": { "avg": 4.7, "count": 120 },
        "value": { "avg": 4.4, "count": 118 }
      },
      "distribution": {
        "participated": { "count": 80, "weight": 3.0, "avg": 4.7 },
        "elsewhere": { "count": 30, "weight": 1.5, "avg": 4.4 },
        "unexperienced": { "count": 18, "weight": 1.0, "avg": 4.2 }
      }
    },
    "myRating": {
      "ratingId": "rt_m042",
      "experienceType": "participated",
      "story": 4.5,
      "gameplay": 4.8,
      "atmosphere": 4.6,
      "value": 4.3,
      "content": "很棒的体验",
      "canEdit": true,
      "canDelete": true,
      "expiresAt": "2025-07-18T23:59:59"
    },
    "items": [
      {
        "ratingId": "rt_m001",
        "userId": "u_10003",
        "nickname": "玩家B",
        "avatarUrl": "https://...",
        "experienceType": "participated",
        "overall": 4.8,
        "dimensions": { "story": 5, "gameplay": 4.8, "atmosphere": 4.8, "value": 4.6 },
        "content": "非常沉浸！",
        "isAnonymous": false,
        "createdAt": "2025-07-09T23:30:00",
        "sessionId": "s_050"
      }
    ],
    "total": 128,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### GET /api/v1/ratings/gm/{gmId}

**接口名称**: GM评分详情  
**功能描述**: 获取GM的完整6维评分统计和评价列表  
**使用端**: 两端  
**权限要求**: 已登录  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| gmId | string | GM用户ID |

**查询参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| sortBy | string | 否 | 排序 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "summary": {
      "overall": 4.7,
      "totalCount": 96,
      "dimensions": {
        "narration": { "avg": 4.8 },
        "improvisation": { "avg": 4.6 },
        "fairness": { "avg": 4.9 },
        "rhythm": { "avg": 4.5 },
        "characterPortrayal": { "avg": 4.7 },
        "atmosphere": { "avg": 4.8 }
      }
    },
    "items": [...],
    "total": 96
  }
}
```

---

### GET /api/v1/ratings/my

**接口名称**: 我的评分列表  
**功能描述**: 获取当前用户的所有评分记录（模组+GM+玩家）  
**使用端**: 小程序端  
**权限要求**: 已登录  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| targetType | string | 否 | 筛选类型：module/gm/player |
| canEdit | bool | 否 | 是否仅显示仍可编辑的 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "ratingId": "rt_m042",
        "targetType": "module",
        "targetId": "m_001",
        "targetName": "诡镇疑云",
        "targetCoverImage": "https://...",
        "overall": 4.55,
        "experienceType": "participated",
        "content": "很棒",
        "canEdit": true,
        "canDelete": true,
        "expiresAt": "2025-07-18T23:59:59",
        "createdAt": "2025-07-15T23:45:00"
      }
    ],
    "total": 25,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### GET /api/v1/admin/ratings

**接口名称**: 评分管理（后台）  
**功能描述**: 管理员查看和管理所有评分，支持违规评分删除  
**使用端**: Web后台  
**权限要求**: module_admin 或 super_admin  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| targetType | string | 否 | 类型筛选 |
| targetId | string | 否 | 被评对象ID |
| userId | string | 否 | 评价者ID |
| isReported | bool | 否 | 是否被举报 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 包含完整的评分记录及用户信息。

---

### DELETE /api/v1/admin/ratings/{ratingId}

**接口名称**: 管理员删除评分  
**功能描述**: 管理员删除违规评分（不受窗口期限制）  
**使用端**: Web后台  
**权限要求**: super_admin  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| ratingId | string | 评分记录ID |

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| reason | string | 是 | 删除原因 |

**返回值**:
```json
{ "code": 0, "message": "评分已删除" }
```

---

## 7. Vote 投票模块

> 对应页面：P13 投票列表、P14 投票详情/参与、P15 发起投票

### POST /api/v1/votes

**接口名称**: 发起投票  
**功能描述**: 创建新的投票活动  
**使用端**: 小程序端  
**权限要求**: gm 角色  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| title | string | 是 | 投票标题（2~50字符） |
| description | string | 否 | 投票说明（最大500字符） |
| candidateModuleIds | string[] | 是 | 候选模组ID数组（2~8个，均须为published状态） |
| voteType | string | 是 | single / multi |
| maxChoices | int | 否 | 多选时最多选几项（voteType=multi时必填，2~候选数） |
| deadline | datetime | 是 | 截止时间（距创建时间1小时~7天） |
| resultVisibility | string | 是 | 结果可见性：always / gm_only / after_vote / after_close |

**业务规则**:
- 候选模组必须都是 published 状态
- 单用户每日最多发起5个投票
- 截止时间必须在创建时间1h~7d之间

**返回值**:
```json
{
  "code": 0,
  "data": {
    "voteId": "v_001",
    "status": "active",
    "shareUrl": "pages/vote/detail?voteId=v_001",
    "message": "投票创建成功"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 60001 | 候选模组数量不符（须2~8个） |
| 60002 | 存在未发布的候选模组 |
| 60003 | 截止时间超出范围 |
| 60004 | 今日发起投票已达上限（5个） |
| 60005 | maxChoices设置不合法 |

---

### GET /api/v1/votes

**接口名称**: 投票列表  
**功能描述**: 分页获取投票列表，支持状态筛选  
**使用端**: 小程序端  
**权限要求**: 已登录  

**请求参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 状态：active/expired/closed |
| createdByMe | bool | 否 | 是否仅看我发起的 |
| participatedByMe | bool | 否 | 是否仅看我参与的 |
| keyword | string | 否 | 搜索关键词 |
| sortBy | string | 否 | 排序：deadline/latest/participants |
| sortOrder | string | 否 | asc/desc |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "voteId": "v_001",
        "title": "下周玩什么？大家来选！",
        "creatorId": "u_10002",
        "creatorName": "GM老王",
        "creatorAvatar": "https://...",
        "voteType": "multi",
        "maxChoices": 3,
        "candidateCount": 5,
        "participantCount": 18,
        "status": "active",
        "deadline": "2025-07-14T22:00:00",
        "myVoteStatus": "not_voted",
        "resultVisibility": "after_close",
        "createdAt": "2025-07-11T10:00:00"
      }
    ],
    "total": 42,
    "page": 1,
    "pageSize": 20
  }
}
```

---

### GET /api/v1/votes/{voteId}

**接口名称**: 投票详情  
**功能描述**: 获取投票完整信息，根据resultVisibility控制结果可见性  
**使用端**: 小程序端  
**权限要求**: 已登录  

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| voteId | string | 投票ID |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "voteId": "v_001",
    "title": "下周玩什么？大家来选！",
    "description": "选出你最想玩的3个模组",
    "creatorId": "u_10002",
    "creatorName": "GM老王",
    "creatorAvatar": "https://...",
    "status": "active",
    "ruleType": "multi",
    "maxChoices": 3,
    "resultVisibility": "after_close",
    "deadline": "2026-07-15T20:00:00+08:00",
    "createdAt": "2026-07-11T10:00:00+08:00",
    "candidates": [
      {
        "candidateId": "vc_001",
        "moduleId": "m_001",
        "moduleName": "克苏鲁的呼唤",
        "moduleCover": "https://...",
        "voteCount": 15,
        "showVoteCount": true
      }
    ],
    "myVote": {
      "hasVoted": true,
      "selectedCandidateIds": ["vc_001", "vc_003"]
    },
    "totalParticipants": 28,
    "canManage": false,
    "isExpired": false
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40401 | 投票不存在 |
| 40402 | 投票已删除 |

---

### POST /api/v1/votes/{voteId}/participate

**接口名称**: 参与投票  
**功能描述**: 用户对投票进行投票，OpenID唯一约束，每人每投票仅投1次  
**使用端**: 小程序端  
**权限要求**: 已登录（玩家/GM均可） |

**路径参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| voteId | string | 是 | 投票ID |

**请求体**:
```json
{
  "selectedCandidateIds": ["vc_001", "vc_003"]
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| selectedCandidateIds | string[] | 是 | 候选模组ID列表；单选时长度=1，多选时长度≤maxChoices |

**返回值**:
```json
{ "code": 0, "message": "投票成功", "data": { "recordId": "vr_001" } }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 投票已截止或已关闭 |
| 40002 | 该用户已投过票（OpenID重复） |
| 40003 | 候选数量超出限制（单选>1或多选>maxChoices） |
| 40004 | 候选模组不属于该投票 |
| 42901 | 今日参与投票次数已达上限(20次) |
| 40401 | 投票不存在 |

---

### POST /api/v1/votes/{voteId}/close

**接口名称**: 提前关闭投票  
**功能描述**: GM提前关闭仍在进行中的投票，关闭后不可再投票  
**使用端**: 小程序端  
**权限要求**: 仅投票创建者(GM) |

**路径参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| voteId | string | 是 | 投票ID |

**返回值**:
```json
{ "code": 0, "message": "投票已关闭" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40301 | 无权操作（非创建者） |
| 40001 | 投票已结束或已关闭 |

---

### POST /api/v1/votes/{voteId}/cancel

**接口名称**: 取消投票  
**功能描述**: GM取消投票（无人参与时可直接取消；有人参与后需确认）  
**使用端**: 小程序端  
**权限要求**: 仅投票创建者(GM) |

**返回值**:
```json
{ "code": 0, "message": "投票已取消" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40301 | 无权操作 |
| 40001 | 投票已结束或已关闭 |

---

### POST /api/v1/votes/{voteId}/create-session

**接口名称**: 根据投票结果一键开团  
**功能描述**: 使用得票最高的候选模组创建开团草稿，自动填充模组和推荐人数区间  
**使用端**: 小程序端  
**权限要求**: 仅投票创建者(GM) |

**路径参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| voteId | string | 是 | 投票ID |

**请求体**:
```json
{
  "candidateId": "vc_001"
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| candidateId | string | 否 | 指定候选模组ID；不传则自动使用最高票候选 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "sessionId": "s_001",
    "moduleId": "m_001",
    "moduleName": "克苏鲁的呼唤",
    "recommendedMinPlayers": 3,
    "recommendedMaxPlayers": 6,
    "status": "draft"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40301 | 无权操作（非创建者） |
| 40001 | 投票尚未截止 |
| 40401 | 投票不存在 |

---

### GET /api/v1/admin/votes

**接口名称**: 后台投票列表  
**功能描述**: 管理员查看所有投票，支持状态筛选  
**使用端**: Web后台  
**权限要求**: super_admin（vote.manage权限） |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 筛选状态: active/closed/cancelled/expired |
| creatorId | string | 否 | 按发起GM筛选 |
| keyword | string | 否 | 搜索标题 |
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页条数，默认20，最大100 |

**返回值**: 分页列表，每项包含投票基本信息、候选数、参与人数、状态

---

### GET /api/v1/admin/votes/{voteId}

**接口名称**: 后台投票详情  
**功能描述**: 管理员查看投票完整详情（含完整结果，不受visibility约束）  
**使用端**: Web后台  
**权限要求**: super_admin（vote.manage权限） |

**返回值**: 完整投票信息 + 所有候选人票数明细 + 参与者列表

---

### POST /api/v1/admin/votes/{voteId}/force-close

**接口名称**: 强制关闭投票  
**功能描述**: 管理员强制关闭任意进行中的投票  
**使用端**: Web后台  
**权限要求*: super_admin（vote.manage权限） |

**返回值**:
```json
{ "code": 0, "message": "投票已被强制关闭" }
```

---

### DELETE /api/v1/admin/votes/{voteId}

**接口名称**: 删除/取消投票  
**功能描述**: 管理员删除投票记录  
**使用端**: Web后台  
**权限要求**: super_admin（vote.manage权限） |

**返回值**:
```json
{ "code": 0, "message": "投票已删除" }
```

---

## 8. Follow 关注模块

> 对应页面：P04 GM主页（关注按钮）、P26 关注列表页、P24 个人主页（关注数展示）、后台 M19 关注数据统计

### POST /api/v1/follows

**接口名称**: 关注GM  
**功能描述**: 当前用户关注指定GM，每个新开团时会收到独立通知  
**使用端**: 小程序端  
**权限要求**: 已登录（任意角色均可关注） |

**请求体**:
```json
{
  "gmId": "gm_001"
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| gmId | string | 是 | 目标GM的ID |

**返回值**:
```json
{ "code": 0, "message": "关注成功", "data": { "followId": "f_001" } }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 不能关注自己 |
| 40002 | 已经关注过了 |
| 40401 | GM不存在或已停用 |
| 42901 | 今日关注/取消操作次数已达上限(50次) |

---

### DELETE /api/v1/follows/{gmId}

**接口名称**: 取消关注GM  
**功能描述**: 取消对指定GM的关注  
**使用端**: 小程序端  
**权限要求**: 已登录 |

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| gmId | string | 目标GM的ID |

**返回值**:
```json
{ "code": 0, "message": "已取消关注" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 未关注该GM |
| 42901 | 今日操作次数已达上限(50次) |

---

### GET /api/v1/follows/gms

**接口名称**: 我的关注列表（我关注的GM）  
**功能描述**: 获取当前用户关注的GM列表，仅自己可见  
**使用端**: 小程序端  
**权限要求**: 已登录 |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页条数，默认20 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "gmId": "gm_001",
        "nickname": "GM老王",
        "avatar": "https://...",
        "isCertified": true,
        "isInternal": false,
        "appointmentType": "offline",
        "location": "上海市浦东新区",
        "sessionCount": 45,
        "avgRating": 4.7,
        "followedAt": "2026-06-01T10:00:00+08:00",
        "activeSessionCount": 2
      }
    ],
    "total": 12
  }
}
```

---

### GET /api/v1/follows/me/followers

**接口名称**: 我的关注者列表  
**功能描述**: GM查看关注自己的用户列表  
**使用端**: 小程序端  
**权限要求**: GM身份 |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 分页列表，每项包含用户头像、昵称、关注时间

---

### GET /api/v1/gms/{gmId}/follow-status

**接口名称**: 查询关注状态  
**功能描述**: 快速检查当前用户是否已关注指定GM（用于GM主页按钮状态）  
**使用端**: 小程序端  
**权限要求**: 已登录 |

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| gmId | string | GM ID |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "isFollowing": true,
    "followerCount": 128
  }
}
```

---

### GET /api/v1/gms/{gmId}/follow-count

**接口名称**: GM关注者数量  
**功能描述**: 公开获取指定GM的关注者数量（无需登录）  
**使用端**: 小程序端  
**权限要求**: 无需登录（公开接口） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "gmId": "gm_001",
    "followerCount": 128
  }
}
```

---

### GET /api/v1/admin/follows/stats

**接口名称**: 关注数据统计  
**功能描述**: 后台关注数据总览：总数/日新增/TOP GM排行  
**使用端*: Web后台  
**权限要求*: super_admin |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| dateRange | string | 否 | 统计日期范围：today/week/month/custom |
| startDate | string | 否 | 自定义起始日期(YYYY-MM-DD) |
| endDate | string | 否 | 自定义结束日期(YYYY-MM-DD) |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "totalFollows": 5680,
    "newToday": 42,
    "newThisWeek": 280,
    "newThisMonth": 1200,
    "topGms": [
      { "gmId": "gm_001", "nickname": "GM老王", "followerCount": 128, "newFollowsWeek": 12 },
      { "gmId": "gm_002", "nickname": "GM阿博", "followerCount": 95, "newFollowsWeek": 8 }
    ]
  }
}
```

---

## 9. Notify 通知模块

> 对应页面：P20 消息中心页、各业务页面消息触发点

### 通知分类枚举

| 分类(category) | 说明 | 触发场景 |
|----------------|------|----------|
| session | 开团相关 | CREATED/UPDATED/RESCHEDULED/CANCELLED/FULL/CONFIRMED/STARTED/COMPLETED |
| registration | 报名相关 | REGISTERED/UNREGISTERED/CONFIRMED/REJECTED |
| rating | 评分相关 | REMINDER/MODIFIED |
| module | 模组相关 | PUBLISHED/OFFLINE/REJECTED |
| management | 管理相关 | GM_CERTIFIED/GM_SUSPENDED/MODULE_REVIEWED |
| follow | 关注相关 | （暂无主动通知） |
| vote | 投票相关 | CLOSED/CANCELLED |
| payment | 支付相关 | PAYMENT_SUCCESS/PAYMENT_FAILED/REFUND |

---

### GET /api/v1/notifications

**接口名称**: 消息列表  
**功能描述**: 获取当前用户的通知消息列表，支持分类筛选和已读/未读筛选  
**使用端*: 小程序端  
**权限要求*: 已登录 |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| readStatus | string | 否 | 筛选已读状态: all/unread/read，默认all |
| category | string | 否 | 分类筛选: session/registration/rating/module/management/vote/payment |
| page | int | 否 | 页码，默认1 |
| pageSize | int | 否 | 每页条数，默认20 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      {
        "notificationId": "n_001",
        "category": "session",
        "type": "CREATED",
        "title": "新开团通知",
        "content": "你关注的GM「老王」发起了新团《克苏鲁的呼唤》，07月15日 14:00开始",
        "summary": 「老王」发起了新团《克苏鲁的呼唤》",
        "targetType": "session",
        "targetId": "s_001",
        "isRead": false,
        "createdAt": "2026-07-11T12:00:00+08:00"
      }
    ],
    "total": 56,
    "unreadCount": 12
  }
}
```

---

### GET /api/v1/notifications/unread-count

**接口名称**: 未读消息数量  
**功能描述**: 快速获取当前用户未读消息总数（用于Tab栏红点）  
**使用端*: 小程序端  
**权限要求*: 已登录 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "totalCount": 12,
    "byCategory": {
      "session": 5,
      "registration": 3,
      "rating": 2,
      "payment": 2
    }
  }
}
```

---

### PUT /api/v1/notifications/{notificationId}/read

**接口名称**: 标记单条已读  
**功能描述**: 将指定消息标记为已读  
**使用端*: 小程序端  
**权限要求*: 已登录 |

**路径参数**:
| 参数名 | type | 说明 |
|--------|------|------|
| notificationId | string | 消息ID |

**返回值**:
```json
{ "code": 0, "message": "已标记为已读" }
```

---

### PUT /api/v1/notifications/read-all

**接口名称**: 全部标记已读  
**功能描述**: 将当前用户所有未读消息批量标记为已读  
**使用端*: 小程序端  
**权限要求*: 已登录 |

**请求体**（可选）:
```json
{
  "category": "session"
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| category | string | 否 | 指定分类全部已读；不传则全部消息标记已读 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "markedCount": 12
  }
}
```

---

### DELETE /api/v1/notifications/clear

**接口名称**: 清空消息记录  
**功能描述**: 清空当前用户的消息记录（按分类或全部清空）  
**使用端*: 小程序端  
**权限要求*: 已登录 |

**请求体**:
```json
{
  "category": "session",
  "beforeDate": "2026-06-01T00:00:00+08:00"
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| category | string | 否 | 按分类清空；不传则清空全部 |
| beforeDate | string | 否 | 清空此日期之前的消息 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "deletedCount": 30
  }
}
```

---

## 10. Venue 场地模块

> 对应页面：P21 包间选择页、P22 支付确认页、P23 支付结果页、A14 包间管理、A15 场地订单管理、A15 前台操作面板

### GET /api/v1/venues/rooms

**接口名称**: 可用包间列表  
**功能描述**: 获取指定日期时段的可用包间列表，含冲突检测和容量信息  
**使用端*: 小程序端 / Web后台  
**权限要求*: 已登录（GM发起开团时调用） |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| date | string | 是 | 日期 YYYY-MM-DD |
| startTime | string | 是 | 开始时间 HH:mm |
| endTime | string | 是 | 结束时间 HH:mm |
| minCapacity | int | 否 | 最小容量过滤（用于人数-容量校验） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "date": "2026-07-15",
    "timeSlot": "14:00-18:00",
    "rooms": [
      {
        "roomId": "r_001",
        "name": "301包间",
        "floor": "3F",
        "capacity": 8,
        "hourlyPrice": 68.00,
        "status": "available",
        "conflictInfo": null,
        "equipment": ["投影仪", "白板"],
        "estimatedCost": 272.00
      },
      {
        "roomId": "r_002",
        "name": "302包间",
        "floor": "3F",
        "capacity": 6,
        "hourlyPrice": 58.00,
        "status": "occupied",
        "conflictInfo": {
          "sessionId": "s_099",
          "gmName": "GM阿博",
          "moduleName": "龙与地下城",
          "time": "14:00-18:00"
        }
      }
    ]
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 日期不能早于今天 |
| 40002 | 时段无效（结束时间必须晚于开始时间） |

---

### GET /api/v1/venues/rooms/{roomId}

**接口名称**: 包间详情  
**功能描述**: 获取单个包间的详细信息  
**使用端*: 小程序端 / Web后台  
**权限要求*: 已登录 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "roomId": "r_001",
    "name": "301包间",
    "floor": "3F",
    "capacity": 8,
    "hourlyPrice": 68.00,
    "status": "active",
    "equipment": ["投影仪", "白板", "音响"],
    "description": "适合6-8人中型跑团",
    "imageUrl": "https://..."
  }
}
```

---

### GET /api/v1/venues/rooms/{roomId}/calendar

**接口名称**: 包间占用日历  
**功能描述**: 获取指定包间在某个月份的占用情况（用于日历视图）  
**使用端*: Web后台  
**权限要求*: super_admin / cashier |

**路径参数**:
| 参数名 | type | 必填 | 说明 |
|--------|------|------|------|
| roomId | string | 是 | 包间ID |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| year | int | 是 | 年份 |
| month | int | 是 | 月份(1-12) |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "roomId": "r_001",
    "year": 2026,
    "month": 7,
    "occupancies": [
      {
        "date": "2026-07-15",
        "slots": [
          { "startTime": "14:00", "endTime": "18:00", "sessionId": "s_001", "gmName": "GM老王", "status": "confirmed" }
        ]
      }
    ]
  }
}
```

---

### POST /api/v1/venues/check-conflict

**接口名称**: 冲突检测（实时校验）  
**功能描述**: 实时检测指定包间在指定时段是否可用（<1s响应）  
**使用端*: 小程序端  
**权限要求*: 已登录 |

**请求体**:
```json
{
  "roomId": "r_001",
  "date": "2026-07-15",
  "startTime": "14:00",
  "endTime": "18:00",
  "excludeSessionId": "s_001"
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| roomId | string | 是 | 包间ID |
| date | string | 是 | 日期 |
| startTime | string | 是 | 开始时间 |
| endTime | string | 是 | 结束时间 |
| excludeSessionId | string | 否 | 排除某Session（编辑开团时用） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "isAvailable": true,
    "conflicts": []
  }
}
```

或冲突时:
```json
{
  "code": 0,
  "data": {
    "isAvailable": false,
    "conflicts": [
      {
        "sessionId": "s_099",
        "gmName": "GM阿博",
        "time": "14:00-18:00",
        "reason": "时段重叠"
      }
    ]
  }
}
```

---

### POST /api/v1/payments/create

**接口名称**: 创建支付订单  
**功能描述**: 非内部GM发布带场地的开团时，创建微信支付订单  
**使用端*: 小程序端  
**权限要求*: GM（is_internal=false） |

**请求体**:
```json
{
  "sessionId": "s_001",
  "roomId": "r_001",
  "date": "2026-07-15",
  "startTime": "14:00",
  "endTime": "18:00"
}
```

**返回值**:
```json
{
  "code": 0,
  "data": {
    "orderId": "pay_001",
    "orderIdDisplay": "TP2026071500001",
    "sessionId": "s_001",
    "roomName": "301包间",
    "date": "2026-07-15",
    "timeSlot": "14:00-18:00",
    "durationHours": 4,
    "unitPrice": 68.00,
    "totalAmount": 272.00,
    "currency": "CNY",
    "wxPayParams": {
      "appId": "wx...",
      "timeStamp": "...",
      "nonceStr": "...",
      "package": "prepay_id=...",
      "signType": "RSA",
      "paySign": "..."
    },
    "expireAt": "2026-07-11T14:35:00+08:00"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 内部GM无需支付 |
| 40002 | Session不存在或不属于当前GM |
| 40003 | Session状态不允许支付（非pending_payment） |
| 40004 | 包间容量不足 |
| 40005 | 包间时段冲突 |

---

### POST /api/v1/payments/{orderId}/callback

**接口名称**: 微信支付回调  
**功能描述**: 接收微信支付异步通知，更新订单状态并发布开团  
**使用端*: 微信服务器回调  
**权限要求*: 微信签名验证 |

**请求体**: 微信支付回调XML/JSON格式

**返回值**: 微信要求的响应格式（{"code":"SUCCESS","message":"OK"}）

**业务逻辑**:
1. 验证微信签名
2. 校验订单金额匹配
3. 更新Payment状态为paid
4. 将Session状态从pending_payment → recruiting
5. 锁定包间时段
6. 发送开团通知给GM关注者
7. 返回success给微信

---

### GET /api/v1/payments/{orderId}/status

**接口名称**: 支付状态查询  
**功能描述**: 查询支付订单当前状态（用于支付结果页轮询）  
**使用端*: 小程序端  
**权限要求*: 订单关联的GM |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "orderId": "pay_001",
    "status": "paid",
    "totalAmount": 272.00,
    "paidAt": "2026-07-11T14:30:00+08:00",
    "transactionId": "420000123420260711...",
    "sessionId": "s_001",
    "sessionStatus": "recruiting"
  }
}
```

---

### POST /api/v1/payments/{orderId}/refund

**接口名称**: 发起退款  
**功能描述**: 触发退款流程（自动重试最多5次，间隔3分钟）  
**使用端*: Web后台 / 系统（内部调用）  
**权限要求*: super_admin / cashier（venue.order.adjust权限） 或系统自动触发 |

**请求体**:
```json
{
  "reason": "GM取消开团",
  "triggeredBy": "session_cancel"
  "operatorId": "admin_001"
}
```

**返回值**:
```json
{
  "code": 0,
  "data": {
    "refundId": "ref_001",
    "status": "processing",
    "amount": 272.00,
    "estimatedCompleteAt": "2026-07-11T14:45:00+08:00"
  }
}
```

**业务规则**:
- 退款自动重试5次，每次间隔3分钟
- 5次均失败后标记failed并通知管理员
- 退款成功后发送站内消息+微信通知给GM

---

### GET /api/v1/payments/{orderId}/refund-status

**接口名称**: 退款状态查询  
**功能描述**: 查询退款进度和结果  
**使用端*: Web后台  
**权限要求*: super_admin / cashier |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "refundId": "ref_001",
    "status": "success",
    "amount": 272.00,
    "retryCount": 2,
    "completedAt": "2026-07-11T14:39:00+08:00",
    "transactionId": "5000000678..."
  }
}
```

---

### GET /api/v1/venues/orders

**接口名称**: 场地订单列表  
**功能描述**: 获取场地付费订单列表  
**使用端*: Web后台  
**权限要求*: super_admin / cashier（venue.order.view权限） |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 状态筛选: pending/paid/refunded/failed/expired |
| gmId | string | 否 | 按GM筛选 |
| roomId | string | 否 | 按包间筛选 |
| dateFrom | string | 否 | 开始日期 |
| dateTo | string | 否 | 结束日期 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 分页列表，每项包含订单号、金额、GM、包间、时段、状态

---

### GET /api/v1/venues/orders/{orderId}

**接口名称**: 场地订单详情  
**功能描述**: 获取单个场地订单的完整信息和操作日志  
**使用端*: Web后台  
**权限要求*: super_admin / cashier |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "orderId": "pay_001",
    "orderNo": "TP2026071500001",
    "sessionId": "s_001",
    "gmId": "gm_001",
    "gmName": "GM老王",
    "roomId": "r_001",
    "roomName": "301包间",
    "date": "2026-07-15",
    "timeSlot": "14:00-18:00",
    "durationHours": 4,
    "unitPrice": 68.00,
    "totalAmount": 272.00,
    "status": "paid",
    "paidAt": "2026-07-11T14:30:00+08:00",
    "refundInfo": null,
    "operationLogs": [
      {
        "logId": "vol_001",
        "type": "payment_success",
        "description": "支付成功，¥272.00",
        "operatorId": "system",
        "operatorName": "系统",
        "createdAt": "2026-07-11T14:30:00+08:00"
      }
    ]
  }
}
```

---

### POST /api/v1/venues/orders/{orderId}/adjust-room

**接口名称**: 前台换包间  
**功能描述**: 线下场景下更换订单的包间，自动计算差价  
**使用端*: Web后台  
**权限要求*: cashier / super_admin（venue.order.adjust权限） |

**请求体**:
```json
{
  "newRoomId": "r_002",
  "reason": "设备故障需要换房间"
}
```

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| newRoomId | string | 是 | 新包间ID |
| reason | string | 否 | 操作原因 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "orderId": "pay_001",
    "oldRoom": { "roomId": "r_001", "name": "301包间", "capacity": 8 },
    "newRoom": { "roomId": "r_002", "name": "302包间", "capacity": 6 },
    "priceDiff": -40.00,
    "priceDiffType": "refund",
    "originalAmount": 272.00,
    "newAmount": 232.00,
    "capacityCheck": {
      "passed": true,
      "message": "新包间容量(6)满足开团最大人数(5)"
    }
  }
}
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40001 | 新包间在此时段已被占用 |
| 40002 | 新包间容量不足（max_players > capacity） |
| 40003 | 订单状态不支持调整 |

---

### POST /api/v1/venues/orders/{orderId}/reschedule

**接口名称**: 前台改期  
**功能描述**: 线下场景下修改订单时间，自动计算差价（不受48h窗口限制）  
**使用端*: Web后台  
**权限要求*: cashier / super_admin（venue.order.adjust权限） |

**请求体**:
```json
{
  "newDate": "2026-07-16",
  "newStartTime": "14:00",
  "newEndTime": "18:00",
  "reason": "客人临时有事"
}
```

**返回值**:
```json
{
  "code": 0,
  "data": {
    "orderId": "pay_001",
    "priceDiff": 0,
    "priceDiffType": "none",
    "operationLogId": "vol_002"
  }
}
```

---

### POST /api/v1/venues/orders/{orderId}/confirm-refund

**接口名称**: 前台确认线下退款  
**功能描述**: 线下已手动完成退款平账，标记订单为已处理  
**使用端*: Web后台  
**权限要求*: cashier / super_admin（venue.order.adjust权限） |

**请求体**:
```json
{
  "actualRefundAmount": 272.00,
  "note": "现金退还，客人已签字确认"
}
```

**返回值**:
```json
{ "code": 0, "message": "退款已确认平账" }
```

---

### GET /api/v1/venues/revenue/stats

**接口名称**: 场地收入统计  
**功能描述**: 场地收入数据汇总（用于仪表盘）  
**使用端*: Web后台  
**权限要求*: super_admin |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| dateRange | string | 否 | today/week/month/year/custom |
| startDate | string | 否 | 自定义起始 |
| endDate | string | 否 | 自定义结束 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "totalRevenue": 58680.00,
    "orderCount": 186,
    "avgOrderAmount": 315.48,
    "refundAmount": 1360.00,
    "netRevenue": 57320.00,
    "utilizationRate": 0.72,
    "dailyBreakdown": [
      { "date": "2026-07-11", "revenue": 1680.00, "orders": 5 }
    ],
    "roomBreakdown": [
      { "roomId": "r_001", "name": "301包间", "revenue": 18560.00, "orders": 62 }
    ]
  }
}
```

---

### GET /api/v1/admin/venues/rooms

**接口名称**: 后台包间列表（管理）  
**功能描述**: 管理员CRUD包间数据  
**使用端*: Web后台  
**权限要求*: super_admin（venue.room.manage权限） |

**查询参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| status | string | 否 | 状态筛选: active/maintenance/disabled |
| keyword | string | 否 | 搜索包间名称 |
| page | int | 否 | 页码 |
| pageSize | int | 否 | 每页条数 |

**返回值**: 分页列表

---

### POST /api/v1/admin/venues/rooms

**接口名称**: 创建包间  
**功能描述**: 管理员新建包间  
**使用端*: Web后台  
**权限要求*: super_admin（venue.room.manage权限） |

**请求体**:
```json
{
  "name": "303包间",
  "floor": "3F",
  "capacity": 10,
  "hourlyPrice": 88.00,
  "equipment": ["投影仪", "白板", "音响", "空调"],
  "description": "大型包间，适合8-10人",
  "status": "active"
}
```

**返回值**:
```json
{
  "code": 0,
  "data": {
    "roomId": "r_005",
    "name": "303包间"
  }
}
```

---

### PUT /api/v1/admin/venues/rooms/{roomId}

**接口名称**: 编辑包间  
**功能描述**: 修改包间信息（价格/容量/状态/设施等）  
**使用端*: Web后台  
**权限要求*: super_admin（venue.room.manage权限） |

**请求体**: 同创建，所有字段可选（部分更新）

**返回值**:
```json
{ "code": 0, "message": "包间信息已更新" }
```

---

### PUT /api/v1/admin/venues/rooms/{roomId}/status

**接口名称**: 修改包间状态  
**功能描述**: 切换包间运营状态（正常/维护中/停用）  
**使用端*: Web后台  
**权限要求*: super_admin（venue.room.manage权限） |

**请求体**:
```json
{
  "status": "maintenance",
  "reason": "空调维修"
}
```

**返回值**:
```json
{ "code": 0, "message": "包间状态已更新" }
```

---

## 11. RBAC 权限模块

> 对应页面：A11 权限管理页、A03-A04 用户管理、A13 GM管理、A05 模组管理等所有后台管理页面

### 权限码设计

**角色定义**:

| 角色(role) | 说明 | 默认权限码 |
|-----------|------|-------------|
| super_admin | 超级管理员 | 全部权限 |
| module_admin | 模组管理员 | module.* (模组全量管理) |
| cashier | 前台收银 | venue.order.*, venue.order.view, venue.order.adjust |
| gm | GM | session.own.*, rating.own*, vote.own*, profile.own* |
| player | 玩家 | rating.own*(对参与的), registration.own*, follow.own* |

**权限码清单(permission codes)**:

| 权限码 | 说明 | 适用角色 |
|--------|------|----------|
| user.manage | 用户管理(CRUD/禁用) | super_admin |
| user.view | 查看用户详情 | super_admin, module_admin |
| module.create | 创建模组 | super_admin, module_admin, certified_gm |
| module.edit | 编辑模组 | super_admin, module_admin, own_gm |
| module.delete | 删除模组 | super_admin |
| module.review | 审核模组 | super_admin, module_admin |
| module.offline | 下架/上架模组 | super_admin, module_admin |
| gm.manage | GM管理(认证/停用/is_internal) | super_admin |
| gm.edit | 编辑GM信息 | super_admin, own_gm |
| session.create | 创建开团 | gm |
| session.own.manage | 管理自己的开团 | gm |
| session.admin.manage | 管理所有开团 | super_admin |
| venue.room.manage | 包间管理 | super_admin |
| venue.order.view | 查看场地订单 | super_admin, cashier |
| venue.order.adjust | 场地订单调整(换包间/改期/退款) | super_admin, cashier |
| rating.own.create | 创建评分 | player, gm(as_player) |
| rating.own.modify | 修改自己的评分 | player, gm(as_player) |
| rating.admin.delete | 删除违规评分 | super_admin |
| vote.manage | 投票管理(强制关闭/取消) | super_admin |
| vote.own.create | 发起投票 | gm |
| rbac.manage | RBAC角色权限管理 | super_admin |
| config.manage | 配置管理(规则/难度/标签/类型) | super_admin |
| security.view | 安全监控查看 | super_admin |

---

### GET /api/v1/auth/permissions

**接口名称**: 当前用户权限列表  
**功能描述**: 获取当前登录用户的角色和完整权限码列表（用于前端路由/按钮级权限控制）  
**使用端*: 两端  
**权限要求*: 已登录 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "userId": "u_10001",
    "roles": ["super_admin"],
    "permissions": [
      "user.manage", "module.*", "gm.manage", "session.admin.manage",
      "venue.*", "rating.admin.delete", "rbac.manage", "config.manage"
    ],
    "isAdmin": true,
    "isGm": false,
    "isCertifiedGm": false,
    "isInternalGm": false
  }
}
```

---

### GET /api/v1/admin/roles

**接口名称**: 角色列表  
**功能描述**: 获取系统中所有角色及其关联的权限码  
**使用端*: Web后台  
**权限要求*: super_admin（rbac.manage权限） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "roles": [
      {
        "roleId": "role_super_admin",
        "roleName": "超级管理员",
        "description": "拥有全部权限",
        "isSystem": true,
        "permissionCodes": ["*"],
        "userCount": 2
      },
      {
        "roleId": "role_module_admin",
        "roleName": "模组管理员",
        "description": "负责模组审核与管理",
        "isSystem": false,
        "permissionCodes": ["module.create", "module.edit", "module.review", "module.offline", "module.view", "user.view"],
        "userCount": 3
      },
      {
        "roleId": "role_cashier",
        "roleName": "前台收银",
        "description": "处理场地订单与线下调整",
        "isSystem": true,
        "permissionCodes": ["venue.order.view", "venue.order.adjust"],
        "userCount": 2
      }
    ]
  }
}
```

---

### PUT /api/v1/admin/roles/{roleId}/permissions

**接口名称**: 修改角色权限  
**功能描述**: 更新指定角色的权限码集合  
**使用端*: Web后台  
**权限要求*: super_admin（rbac.manage权限） |

**请求体**:
```json
{
  "permissionCodes": ["module.create", "module.edit", "module.review", "module.offline", "user.view", "config.manage"]
}
```

**返回值**:
```json
{ "code": 0, "message": "角色权限已更新" }
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 40301 | 不能修改系统内置角色 |
| 40001 | 权限码无效 |

---

### PUT /api/v1/admin/users/{userId}/roles

**接口名称**: 分配用户角色  
**功能描述**: 为用户分配或移除角色（支持多角色）  
**使用端*: Web后台  
**权限要求*: super_admin（rbac.manage权限） |

**请求体**:
```json
{
  "roles": ["super_admin", "module_admin"]
}
```

**返回值**:
```json
{ "code": 0, "message": "用户角色已更新" }
```

---

### GET /api/v1/admin/config/rule-systems

**接口名称**: 规则系统配置列表  
**功能描述**: 获取规则系统配置项（用于模组录入下拉选择）  
**使用端*: Web后台 / 小程序端(只读)  
**权限要求*: 后台写操作需config.manage；小程序端只读公开 |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "items": [
      { "id": "rs_001", "name": "D&D 5E", "sortOrder": 1, "isActive": true, "moduleCount": 25 },
      { "id": "rs_002", "name": "COC 7E", "sortOrder": 2, "isActive": true, "moduleCount": 18 }
    ]
  }
}
```

---

### POST /api/v1/admin/config/rule-systems
**PUT /api/v1/admin/config/rule-systems/{id}**
**DELETE /api/v1/admin/config/rule-systems/{id}**

**接口名称**: 规则系统 CRUD  
**功能描述**: 管理规则系统配置项  
**使用端*: Web后台  
**权限要求*: super_admin（config.manage权限） |

---

### GET /api/v1/admin/config/difficulty-levels

**接口名称**: 难度等级配置列表  
**功能描述**: 获取难度等级配置项  
**使用端*: Web后台 / 小程序端(只读)  
**权限要求*: 写操作需config.manage |

同上格式，CRUD接口。

---

### GET /api/v1/admin/config/orientation-tags

**接口名称**: 偏向标签配置列表  
**功能描述**: 获取偏向标签配置项  
**使用端*: Web后台 / 小程序端(只读)  
**权限要求*: 写操作需config.manage |

同上格式，CRUD接口。

---

### GET /api/v1/admin/config/module-types

**接口名称**: 模组类型配置列表  
**功能描述**: 获取模组类型配置项  
**使用端*: Web后台 / 小程序端(只读)  
**权限要求*: 写操作需config.manage |

同上格式，CRUD接口。

---

### GET /api/v1/public/config

**接口名称**: 公开配置聚合接口  
**功能描述**: 一次性获取所有前端需要的配置项（规则系统/难度等级/偏向标签/模组类型/发行年份选项），减少请求数  
**使用端*: 小程序端  
**权限要求*: 无需登录（公开接口） |

**返回值**:
```json
{
  "code": 0,
  "data": {
    "ruleSystems": [...],
    "difficultyLevels": [...],
    "orientationTags": [...],
    "moduleTypes": [...],
    "yearOptions": ["2026", "2025", "2024", "2023", "2022", "2021以前", "不确定"],
    "appointmentTypes": [{"value":"online","label":"线上约团"},{"value":"offline","label":"线下约团"},{"value":"both","label":"均可"}],
    "systemSettings": {
      "paymentTimeoutMinutes": 30,
      "ratingModifyWindowExperiencedHours": 72,
      "ratingModifyWindowOtherHours": 24,
      "rescheduleLimitPerSession": 1,
      "rescheduleWindowHours": 48,
      "followDailyLimit": 50,
      "voteDailyLimit": 20,
      "voteMinCandidates": 2,
      "voteMaxCandidates": 8,
      "voteMinDeadlineHours": 1,
      "voteMaxDeadlineHours": 168
    }
  }
}
```

---

## 附录A：全局错误码表

| 错误码 | HTTP状态码 | 说明 |
|--------|-----------|------|
| 0 | 200 | 成功 |
| 40001 | 400 | 请求参数错误 |
| 40002 | 400 | 业务规则校验失败 |
| 40003 | 400 | 数据状态冲突（如并发修改） |
| 40101 | 401 | 未登录或Token过期 |
| 40301 | 403 | 无权限执行此操作 |
| 40401 | 404 | 资源不存在 |
| 40402 | 404 | 资源已删除 |
| 40901 | 409 | 资源冲突（如重复报名） |
| 42901 | 429 | 操作频率超限 |
| 50001 | 500 | 服务器内部错误 |
| 50002 | 503 | 第三方服务不可用（如微信支付） |

### 业务专属错误码

| 错误码 | 说明 |
|--------|------|
| SESSION_STATUS_INVALID | Session状态不允许此操作 |
| SESSION_NOT_OWNER | 不是该Session的创建者 |
| SESSION_FULL | Session已满员 |
| SESSION_EXPIRED | Session报名/支付已超时 |
| ROOM_CAPACITY_EXCEEDED | 开团人数超过包间容量 |
| ROOM_CONFLICT | 包间时段冲突 |
| RESCHEDULE_WINDOW_VIOLATED | 违反48h改期窗口 |
| RESCHEDULE_LIMIT_EXCEEDED | 改期次数已用尽 |
| CANCEL_WINDOW_VIOLATED | 违反48h取消窗口 |
| MODULE_NOT_PUBLISHED | 模组不是published状态 |
| MODULE_OFFLINE | 模组已下架 |
| RATING_NOT_QUALIFIED | 无评分资格 |
| RATING_WINDOW_CLOSED | 评分修改窗口已过 |
| RATING_DUPLICATE | 重复评分 |
| VOTE_EXPIRED | 投票已截止 |
| VOTE_ALREADY_PARTICIPATED | 已参与过该投票 |
| PAYMENT_TIMEOUT | 支付超时 |
| PAYMENT_FAILED | 支付失败 |
| REFUND_FAILED | 退款失败 |
| SENSITIVE_WORD_DETECTED | 敏感词命中 |
| GM_NOT_CERTIFIED | GM未认证（无法上传模组） |

---

## 附录B：接口统计总览

| 模块 | 小程序端接口 | Web后台接口 | 公共/系统接口 | 合计 |
|------|------------|------------|--------------|------|
| 1.User 用户模块 | 6 | 4 | 2 | 12 |
| 2.Module 模组模块 | 6 | 7 | 2 | 15 |
| 3.GM GM模块 | 5 | 5 | 1 | 11 |
| 4.Session 开团模块 | 10 | 3 | 0 | 13 |
| 5.Rating 评分模块 | 5 | 3 | 1 | 9 |
| 6.Vote 投票模块 | 6 | 4 | 0 | 10 |
| 7.Follow 关注模块 | 5 | 1 | 1 | 7 |
| 8.Notify 通知模块 | 5 | 0 | 0 | 5 |
| 9.Venue 场地模块 | 4 | 10 | 1 | 15 |
| 10.RBAC 权限模块 | 1 | 8 | 2 | 11 |
| **合计** | **53** | **45** | **10** | **108** |

---

## 附录C：页面-接口映射速查表

### 小程序端核心页面

| 页面 | 主要依赖接口 |
|------|-------------|
| P01 首页 | GET /sessions(recruiting/full) + GET /notifications/unread-count + GET /public/config |
| P02 模组列表 | GET /modules + GET /public/config(筛选项) |
| P03 模组详情 | GET /modules/{id} + GET /ratings/module/{id} + GET /sessions?moduleId= |
| P04 GM主页 | GET /gms/{id} + GET /gms/{id}/follow-status + GET /ratings/gm/{id} |
| P05 GM信息编辑 | PUT /gms/profile |
| P06 开团列表 | GET /sessions + GET /sessions/mine/gm + GET /sessions/mine/player |
| P07 开团详情 | GET /sessions/{id} + GET /sessions/{id}/registrations |
| P08 发起开团 | POST /sessions + GET /venues/rooms + POST /payments/create |
| P09 编辑开团 | PUT /sessions/{id} |
| P10 报名管理 | GET /sessions/{id}/registrations + POST .../confirm + POST .../cancel |
| P11 改期确认 | POST /sessions/{id}/reschedule |
| P12 评分中心 | GET /ratings/my |
| P13-P15 评分表单 | POST /ratings/module + POST /ratings/gm + POST /ratings/player |
| P16 评分记录 | GET /ratings/my |
| P17-P19 投票 | GET/POST /votes + POST .../participate + POST .../create-session |
| P20 消息中心 | GET /notifications + PUT .../read-all |
| P21 包间选择 | GET /venues/rooms + POST /check-conflict |
| P22-P23 支付 | POST /payments/create + GET /payments/{id}/status |
| P24-P28 个人中心 | GET /users/me + GET /follows/gms + GET /sessions/mine/player |

### Web后台核心页面

| 页面 | 主要依赖接口 |
|------|-------------|
| A01 登录 | POST /auth/login |
| A02 仪表盘 | 多个stats接口聚合 |
| A03-A04 用户管理 | GET/PUT /admin/users |
| A05 模组列表 | GET /admin/modules |
| A06 模组编辑 | POST/PUT /admin/modules |
| A07 审核队列 | GET /admin/modules?status=pending + PUT .../review |
| A08 GM管理 | GET /admin/gms + PUT .../certify |
| A09-A10 包间管理 | CRUD /admin/venues/rooms + GET .../calendar |
| A11-A12 订单管理 | GET /venues/orders + GET .../{id} + POST .../adjust-* |
| A13 前台操作面板 | 同上 + refund接口 |
| A14-A16 配置管理 | CRUD /admin/config/* |
| A17 权限管理 | GET/PUT /admin/roles + PUT /admin/users/{id}/roles |
| A18-A20 投票/关注/安全 | 各自admin接口 |

---

*文档生成时间: 2026-07-11  *  总计 **108个RESTful API接口**，覆盖小程序端28页面 + Web后台20页面的全部数据需求。