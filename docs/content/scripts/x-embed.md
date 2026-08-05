---
title: X 嵌入
description: 在服务器端渲染 X 帖子，无需直接从浏览器向 X 请求帖子 JSON 或代理图片。
links:
  - label: ScriptXEmbed
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptXEmbed.vue
    size: xs
---

[X（前身为 Twitter）](https://x.com) 是一个用于分享帖子的社交媒体平台。

[`<ScriptXEmbed>`{lang="html"}](/scripts/x-embed){lang="html"} 通过你的 Nuxt 服务器获取帖子数据，并通过插槽将其公开。帖子 JSON 和图片会通过你的源站传递，而不是加载 X 的小组件 JavaScript。

::script-stats
::

::script-docs{embed}
::

这会注册所需的服务器 API 路由（`/_scripts/embed/x` 和 `/_scripts/embed/x-image`），用于处理推文数据的获取和图片代理。

## [`<ScriptXEmbed>`{lang="html"}](/scripts/x-embed){lang="html"}

帖子端点会缓存联合响应 10 分钟，并通过图片端点重写个人资料照片、附加照片、实体媒体、引用帖子图片和视频海报。视频变体 URL 保持不变，因此在 `<video>`{lang="html"} 元素中渲染视频变体时，浏览器会直接向 X 发起请求。

::callout{color="amber"}
图片代理会验证初始主机名，但目前会在不验证每个目标的情况下跟随重定向。在重定向目标也经过检查之前，不要将该允许列表视为完整的 SSRF 边界。
::

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

```vue [样式化推文卡片]
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
  userAvatar: string // 代理后的 URL
  isVerified: boolean | undefined
  // 推文内容
  text: string
  // 格式化后数据
  datetime: string // "12:47 PM · Feb 5, 2024"
  createdAt: Date
  likes: number
  likesFormatted: string // "1.2K"
  replies: number
  repliesFormatted: string // "234"
  // 媒体
  photos?: Array<NonNullable<XEmbedTweetData['photos']>[number] & {
    proxiedUrl: string
  }>
  video: {
    poster: string
    posterProxied: string
    variants: Array<{ type: string, src: string }>
  } | null
  // 链接
  tweetUrl: string
  userUrl: string
  // 引用推文
  quotedTweet?: XEmbedTweetData
  // 回复上下文
  isReply: boolean
  replyToUser?: string
  // 辅助方法
  proxyImage: (imageUrl: string) => string
}
```

### 命名插槽

| 插槽      | 描述                                      |
|-----------|-------------------------------------------|
| `default` | 主内容，带插槽属性                        |
| `loading` | 获取推文数据时展示                         |
| `error`   | 推文数据获取失败时展示，接收 `{ error }`  |

## 数据流

该实现遵循 [Cloudflare Zaraz 的服务器渲染嵌入方法](https://blog.cloudflare.com/zaraz-supports-server-side-rendering-of-embeds/)。页面中不会运行任何 X JavaScript，并且 X 不会收到访问者的 IP 地址，也不会收到帖子 JSON 或代理图片的请求。渲染后的视频变体以及指向 X 的链接仍会直接联系 X。

::script-types
::