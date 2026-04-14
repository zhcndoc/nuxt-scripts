---
title: X 嵌入
description: 服务端渲染的 X（Twitter）嵌入，零客户端 API 调用。
links:
  - label: ScriptXEmbed
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptXEmbed.vue
    size: xs
---

[X（前身为 Twitter）](https://x.com) 是一个用于分享帖子的社交媒体平台。

Nuxt Scripts 提供了 [`<ScriptXEmbed>`{lang="html"}](/scripts/x-embed){lang="html"} 组件，它在服务器端获取推文数据，并通过插槽暴露出来，实现完全的样式控制。所有数据均通过你的服务器代理 —— 不进行任何客户端对 X 的 API 调用。

::script-stats
::

::script-docs{embed}
::

::callout{type="info"}
This script's proxy endpoints use [HMAC URL signing](/docs/guides/first-party#proxy-endpoint-security) when you configure a `NUXT_SCRIPTS_PROXY_SECRET`. See the [security guide](/docs/guides/first-party#proxy-endpoint-security) for setup instructions.
::

This registers the required server API routes (`/_scripts/embed/x` and `/_scripts/embed/x-image`) that handle fetching tweet data and proxying images.

## [`<ScriptXEmbed>`{lang="html"}](/scripts/x-embed){lang="html"}

`<ScriptXEmbed>` 组件是一个无头组件，功能包括：
- 通过 X 联合 API 在服务器端获取推文数据
- 通过你的服务器代理所有图片以保护隐私
- 通过作用域插槽暴露推文数据，实现自定义渲染
- 缓存响应 10 分钟

### 演示

::code-group

:x-embed-demo{label="输出"}

```vue [基础用法]
<template>
  <ScriptXEmbed tweet-id="1754336034228171055">
    <template #default="{ userName, userHandle, text, datetime, likesFormatted }">
      <div class="border rounded-lg p-4 max-w-md">
        <p class="font-bold">
          {{ userName }} (@{{ userHandle }})
        </p>
        <p>{{ text }}</p>
        <p class="text-gray-500 text-sm">{{ datetime }} - {{ likesFormatted }} 赞</p>
      </div>
    </template>
  </ScriptXEmbed>
</template>
```

<template [样式化推文卡片]>
<template>
  <ScriptXEmbed tweet-id="1754336034228171055">
    <template #default="{ userName, userHandle, userAvatar, text, datetime, likesFormatted, repliesFormatted, photos, isVerified }">
      <div class="max-w-lg bg-white dark:bg-gray-800 rounded-xl border p-4">
        <!-- 头部 -->
        <div class="flex items-start gap-3 mb-3">
          <img :src="userAvatar" :alt="userName" class="w-12 h-12 rounded-full">
          <div>
            <span class="font-bold">{{ userName }}</span>
            <span v-if="isVerified" class="text-blue-500 ml-1">✓</span>
            <p class="text-gray-500">
              @{{ userHandle }}
            </p>
          </div>
        </div>
        <!-- 内容 -->
        <p class="mb-3 whitespace-pre-wrap">{{ text }}</p>
        <!-- 图片 -->
        <div v-if="photos?.length" class="mb-3 rounded-xl overflow-hidden">
          <img v-for="photo in photos" :key="photo.url" :src="photo.proxiedUrl" class="w-full">
        </div>
        <!-- 底部 -->
        <div class="flex items-center gap-4 text-gray-500 text-sm">
          <span>{{ datetime }}</span>
          <span>{{ repliesFormatted }} 条回复</span>
          <span>{{ likesFormatted }} 赞</span>
        </div>
      </div>
    </template>

    <template #loading>
      <div class="animate-pulse bg-gray-100 rounded-xl p-4 max-w-lg">
        加载推文中...
      </div>
    </template>

    <template #error>
      <div class="bg-red-50 border border-red-200 rounded-xl p-4 max-w-lg">
        加载推文失败
      </div>
    </template>
  </ScriptXEmbed>
</template>
```

::

### 属性

`ScriptXEmbed` 组件接受以下属性：

| 属性               | 类型             | 默认值                    | 描述                 |
|--------------------|------------------|---------------------------|----------------------|
| `tweetId`          | `string`         | 必填                      | 要嵌入的推文 ID       |
| `apiEndpoint`      | `string`         | `/api/_scripts/x-embed`   | 获取推文数据的自定义 API 端点 |
| `imageProxyEndpoint` | `string`       | `/api/_scripts/x-embed-image` | 代理图片的自定义端点     |
| `rootAttrs`        | `HTMLAttributes` | `{}`                      | 根元素属性             |

### 插槽属性

默认插槽接收以下属性：

```ts
interface SlotProps {
  // 原始数据
  tweet: XEmbedTweetData
  // 用户信息
  userName: string
  userHandle: string
  userAvatar: string // 代理 URL
  userAvatarOriginal: string // X 原始 URL
  isVerified: boolean
  // 推文内容
  text: string
  // 格式化后数据
  datetime: string // "12:47 PM · Feb 5, 2024"
  createdAt: Date
  likes: number
  likesFormatted: string // "1.2K"
  replies: number
  repliesFormatted: string // "234"
  // 媒体资源
  photos?: Array<{
    URL: string
    proxiedUrl: string
    width: number
    height: number
  }>
  video?: {
    poster: string
    posterProxied: string
    variants: Array<{ type: string, src: string }>
  }
  // Links
  tweetUrl: string
  userUrl: string
  // Quote tweet
  quotedTweet?: XEmbedTweetData
  // 回复上下文
  isReply: boolean
  replyToUser?: string
  // 辅助函数
  proxyImage: (URL: string) => string
}
```

### 命名插槽

| 插槽      | 描述                                      |
|-----------|-------------------------------------------|
| `default` | 主内容，带插槽属性                        |
| `loading` | 获取推文数据时展示                         |
| `error`   | 推文数据获取失败时展示，接收 `{ error }`  |

## 工作原理

1. **服务器端获取**：在服务器端渲染时从 `cdn.syndication.twimg.com` 获取推文数据
2. **图片代理**：所有图片 URL 都被重写，通过 `/_scripts/embed/x-image` 代理
3. **缓存**：响应在服务器层缓存 10 分钟
4. **无客户端 API 调用**：用户浏览器不会直接联系 X

此方案灵感来源于 [Cloudflare Zaraz 的嵌入实现](https://blog.cloudflare.com/zaraz-supports-server-side-rendering-of-embeds/)。

## 隐私优势

- 不加载第三方 JavaScript
- X 不设置 Cookie
- 浏览器与 X 无直接通信
- 用户 IP 地址不会与 X 共享
- 所有内容均从你的域名提供

::script-types
::