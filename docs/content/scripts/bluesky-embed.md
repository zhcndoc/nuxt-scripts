---
title: Bluesky 嵌入
description: 无需浏览器向 Bluesky 发起请求的服务端渲染 Bluesky 嵌入内容。
links:
  - label: ScriptBlueskyEmbed
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptBlueskyEmbed.vue
    size: xs
---

[Bluesky](https://bsky.app) 是一个基于 [AT Protocol](https://atproto.com/) 构建的去中心化社交媒体平台。

[`<ScriptBlueskyEmbed>`{lang="html"}](/scripts/bluesky-embed){lang="html"} 通过你的 Nuxt 服务器获取帖子数据，并提供带作用域的插槽，因此标记和样式都保留在你的应用中。上游请求使用 Bluesky 的 [`getPostThread`](https://docs.bsky.app/docs/tutorials/viewing-threads) API。

::script-stats
::

::script-docs{embed}
::

启用此集成后，会注册用于获取帖子数据的 `/_scripts/embed/bluesky` 和用于获取图片的 `/_scripts/embed/bluesky-image`。

## [`<ScriptBlueskyEmbed>`{lang="html"}](/scripts/bluesky-embed){lang="html"}

帖子端点会缓存线程响应 10 分钟，并缓存句柄到 DID 的查找结果 24 小时。它会通过图像端点重写头像、帖子图片和外部卡片缩略图。该组件还会将富文本 facet 转换为链接，供 `richText` 插槽 prop 使用。

::callout{color="amber"}
`richText` 是经过转义的 HTML，但当前的 facet 转换器不会限制链接 URI 方案。如果你嵌入的帖子并非由你控制，请渲染纯文本 `text` prop，或使用拒绝 `javascript:` 等方案的允许列表对 `richText` 进行清理。
::

::callout{color="amber"}
图像代理会验证初始主机名，但目前会跟随重定向而不会验证每个目标。在同时检查重定向目标之前，不要将该允许列表视为完整的 SSRF 防护边界。
::

### 示例

::code-group

:bluesky-embed-demo{label="输出"}

```vue [基础用法]
<template>
  <ScriptBlueskyEmbed post-url="https://bsky.app/profile/bsky.app/post/3mgnwwvj3u22a">
    <template #default="{ displayName, handle, text, datetime, likesFormatted }">
      <div class="border rounded-lg p-4 max-w-md">
        <p class="font-bold">
          {{ displayName }} (@{{ handle }})
        </p>
        <p>{{ text }}</p>
        <p class="text-gray-500 text-sm">
          {{ datetime }} · {{ likesFormatted }} 点赞
        </p>
      </div>
    </template>
  </ScriptBlueskyEmbed>
</template>
```

```vue [Bluesky 卡片（Tailwind）]
<template>
  <ScriptBlueskyEmbed post-url="https://bsky.app/profile/bsky.app/post/3mgnwwvj3u22a">
    <template #default="{ displayName, handle, avatar, text, datetime, likes, likesFormatted, reposts, repostsFormatted, replies, images, externalEmbed, postUrl, authorUrl }">
      <div class="max-w-[600px] bg-white dark:bg-[#151d28] rounded-2xl border border-gray-200 dark:border-[#2c3a4e] font-sans text-[15px]">
        <!-- 头部 -->
        <div class="flex items-center gap-3 px-4 pt-4 pb-3">
          <a :href="authorUrl" target="_blank" rel="noopener noreferrer" class="shrink-0">
            <img :src="avatar" :alt="displayName" class="w-[42px] h-[42px] rounded-full bg-gray-100 dark:bg-[#1c2736] ring-1 ring-black/5 dark:ring-white/10">
          </a>
          <a :href="authorUrl" target="_blank" rel="noopener noreferrer" class="min-w-0 no-underline">
            <div class="font-semibold text-gray-900 dark:text-white truncate leading-snug">{{ displayName }}</div>
            <div class="text-gray-500 dark:text-[#abb8c9] text-[13px] truncate leading-snug">@{{ handle }}</div>
          </a>
          <!-- Bluesky 蝴蝶图标 -->
          <a :href="postUrl" target="_blank" rel="noopener noreferrer" class="ml-auto shrink-0 text-[#1185fe] hover:text-[#0a6fd4]" aria-label="在 Bluesky 查看">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 568 501" fill="currentColor">
              <path d="M123.121 33.664C188.241 82.553 258.281 181.68 284 234.873c25.719-53.192 95.759-152.32 160.879-201.21C491.866-1.611 568-28.906 568 57.947c0 17.346-9.945 145.713-15.778 166.555c-20.275 72.453-94.155 90.933-159.875 79.748c114.875 19.831 144.097 85.561 81.022 151.291C363.929 569.326 289.18 462.062 284 449.7c-.36-.86-.36-.86 0 0c-5.18 12.362-79.929 119.626-189.369 5.84c-63.075-65.729-33.853-131.46 81.022-151.29c-65.72 11.184-139.6-7.296-159.875-79.749C9.945 203.659 0 75.291 0 57.946C0-28.906 76.134-1.612 123.121 33.664" />
            </svg>
          </a>
        </div>

        <!-- 内容 -->
        <div class="px-4 pb-2">
          <div class="text-gray-900 dark:text-white whitespace-pre-wrap break-words leading-[22px]">
            {{ text }}
          </div>
        </div>

        <!-- 图片 -->
        <div v-if="images?.length" class="px-4 pb-2">
          <div class="rounded-xl overflow-hidden border border-gray-200 dark:border-[#2c3a4e]" :class="images.length > 1 ? 'grid grid-cols-2 gap-0.5' : ''">
            <img v-for="(img, i) in images" :key="i" :src="img.fullsize" :alt="img.alt" class="w-full object-cover" :class="images.length > 1 ? 'aspect-square' : ''">
          </div>
        </div>

        <!-- 外部嵌入卡片 -->
        <div v-if="externalEmbed" class="px-4 pb-2">
          <a :href="externalEmbed.uri" target="_blank" rel="noopener noreferrer" class="block rounded-xl border border-gray-200 dark:border-[#2c3a4e] overflow-hidden no-underline hover:bg-gray-50 dark:hover:bg-[#1c2736] transition-colors">
            <img v-if="externalEmbed.thumb" :src="externalEmbed.thumb" :alt="externalEmbed.title" class="w-full aspect-video object-cover">
            <div class="px-3 py-2">
              <div class="font-semibold text-gray-900 dark:text-white text-[15px] leading-5 line-clamp-2">{{ externalEmbed.title }}</div>
              <div class="text-gray-500 dark:text-[#abb8c9] text-[13px] leading-[17px] line-clamp-2 mt-0.5">{{ externalEmbed.description }}</div>
              <div class="flex items-center gap-1 mt-1 text-[11px] text-gray-400 dark:text-[#abb8c9]">
                <svg fill="none" viewBox="0 0 24 24" width="12" height="12"><path fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" d="M4.4 9.493C4.14 10.28 4 11.124 4 12a8 8 0 1 0 10.899-7.459l-.953 3.81a1 1 0 0 1-.726.727l-3.444.866-.772 1.533a1 1 0 0 1-1.493.35L4.4 9.493Zm.883-1.84L7.756 9.51l.44-.874a1 1 0 0 1 .649-.52l3.306-.832.807-3.227a7.993 7.993 0 0 0-7.676 3.597ZM2 12C2 6.477 6.477 2 12 2s10 4.477 10 10-4.477 10-10 10S2 17.523 2 12Zm8.43.162a1 1 0 0 1 .77-.29l1.89.121a1 1 0 0 1 .494.168l2.869 1.928a1 1 0 0 1 .336 1.277l-.973 1.946a1 1 0 0 1-.894.553h-2.92a1 1 0 0 1-.831-.445L9.225 14.5a1 1 0 0 1 .126-1.262l1.08-1.076Z" /></svg>
                {{ externalEmbed.uri }}
              </div>
            </div>
          </a>
        </div>

        <!-- 时间戳 -->
        <div class="px-4 pt-1 pb-3">
          <a :href="postUrl" target="_blank" rel="noopener noreferrer" class="text-[13px] text-gray-500 dark:text-[#abb8c9] no-underline hover:underline">
            {{ datetime }}
          </a>
        </div>

        <!-- 互动统计 -->
        <div v-if="likes || reposts || replies" class="flex items-center gap-4 px-4 py-3 border-t border-gray-200 dark:border-[#2c3a4e] text-[15px]">
          <a v-if="likes" :href="`${postUrl}/liked-by`" target="_blank" rel="noopener noreferrer" class="no-underline hover:underline text-gray-500 dark:text-[#abb8c9]">
            <span class="font-semibold text-gray-900 dark:text-white">{{ likesFormatted }}</span> 点赞
          </a>
          <span v-if="reposts" class="text-gray-500 dark:text-[#abb8c9]">
            <span class="font-semibold text-gray-900 dark:text-white">{{ repostsFormatted }}</span> 转发
          </span>
        </div>
      </div>
    </template>

    <template #loading>
      <div class="max-w-[600px] bg-white dark:bg-[#151d28] rounded-2xl border border-gray-200 dark:border-[#2c3a4e] p-4">
        <div class="animate-pulse flex gap-3">
          <div class="w-[42px] h-[42px] rounded-full bg-gray-200 dark:bg-[#2c3a4e]" />
          <div class="flex-1 space-y-2 py-1">
            <div class="h-4 bg-gray-200 dark:bg-[#2c3a4e] rounded w-1/3" />
            <div class="h-3 bg-gray-200 dark:bg-[#2c3a4e] rounded w-1/4" />
          </div>
        </div>
        <div class="animate-pulse mt-3 space-y-2">
          <div class="h-4 bg-gray-200 dark:bg-[#2c3a4e] rounded w-full" />
          <div class="h-4 bg-gray-200 dark:bg-[#2c3a4e] rounded w-5/6" />
          <div class="h-4 bg-gray-200 dark:bg-[#2c3a4e] rounded w-2/3" />
        </div>
      </div>
    </template>

    <template #error>
      <div class="max-w-[600px] bg-white dark:bg-[#151d28] rounded-2xl border border-gray-200 dark:border-[#2c3a4e] p-4 text-center text-gray-500 dark:text-[#abb8c9]">
        <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 568 501" fill="currentColor" class="mx-auto mb-2 opacity-30"><path d="M123.121 33.664C188.241 82.553 258.281 181.68 284 234.873c25.719-53.192 95.759-152.32 160.879-201.21C491.866-1.611 568-28.906 568 57.947c0 17.346-9.945 145.713-15.778 166.555c-20.275 72.453-94.155 90.933-159.875 79.748c114.875 19.831 144.097 85.561 81.022 151.291C363.929 569.326 289.18 462.062 284 449.7c-.36-.86-.36-.86 0 0c-5.18 12.362-79.929 119.626-189.369 5.84c-63.075-65.729-33.853-131.46 81.022-151.29c-65.72 11.184-139.6-7.296-159.875-79.749C9.945 203.659 0 75.291 0 57.946C0-28.906 76.134-1.612 123.121 33.664" /></svg>
        加载帖子失败
      </div>
    </template>
  </ScriptBlueskyEmbed>
</template>
```

```vue [极简]
<template>
  <ScriptBlueskyEmbed post-url="https://bsky.app/profile/bsky.app/post/3mgnwwvj3u22a">
    <template #default="{ displayName, handle, text, datetime, likesFormatted }">
      <div class="border rounded-lg p-4 max-w-md">
        <p class="font-bold">
          {{ displayName }} (@{{ handle }})
        </p>
        <p>{{ text }}</p>
        <p class="text-gray-500 text-sm">
          {{ datetime }} · {{ likesFormatted }} 点赞
        </p>
      </div>
    </template>
  </ScriptBlueskyEmbed>
</template>
```

::

### 插槽 Props

默认插槽接收以下属性：

```ts
interface SlotProps {
  // 原始数据
  post: BlueskyEmbedPostData
  // 作者信息
  displayName: string
  handle: string
  avatar: string // 代理 URL
  isVerified: boolean
  // 帖子内容
  text: string // 纯文本
  richText: string // 含链接、提及和标签的 HTML
  langs?: string[] // 语言代码
  // 格式化数值
  datetime: string // "12:47 PM · 2024 年 2 月 5 日"
  createdAt: Date
  likes: number
  likesFormatted: string // "1.2K"
  reposts: number
  repostsFormatted: string // "234"
  replies: number
  repliesFormatted: string // "42"
  quotes: number
  quotesFormatted: string // "12"
  // 媒体
  images?: Array<{
    thumb: string // 代理缩略图 URL
    fullsize: string // 代理全尺寸 URL
    alt: string
    aspectRatio?: { width: number, height: number }
  }>
  externalEmbed?: {
    uri: string
    title: string
    description: string
    thumb?: string // 代理 URL
  }
  // 链接
  postUrl: string
  authorUrl: string
  // 辅助函数
  proxyImage: (url: string) => string
}
```

### 命名插槽

| 插槽 | 描述 |
|------|-------------|
| `default` | 主内容，使用插槽属性渲染 |
| `loading` | 数据加载中显示 |
| `error` | 获取失败时显示，接收 `{ error }` |

## 数据流

页面中不会运行 Bluesky JavaScript。帖子 JSON 和图片会从你的源站发送到浏览器，因此 Bluesky 不会在这些请求中获取访客的 IP 地址。渲染内容中的链接在点击后仍会打开 Bluesky。

## 作者选择退出

如果帖子或作者带有 Bluesky 的 [`!no-unauthenticated` 标签](https://docs.bsky.app/docs/advanced-guides/moderation#global-label-values)，该端点会以 403 响应拒绝请求，并且组件会显示错误插槽。该标签表示，在支持该标签的客户端中，已登出用户不应看到相关内容；其适用范围比仅针对嵌入内容的偏好更广泛。

::script-types
::
