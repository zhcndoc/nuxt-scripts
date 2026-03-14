---

title: Bluesky 嵌入
description: 服务器端渲染的 Bluesky 嵌入，无需客户端 API 调用。
links:
  - label: ScriptBlueskyEmbed
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptBlueskyEmbed.vue
    size: xs

---

[Bluesky](https://bsky.app) 是一个基于 AT Protocol 构建的去中心化社交媒体平台。

Nuxt Scripts 提供了一个 [`<ScriptBlueskyEmbed>`{lang="html"}](/scripts/bluesky-embed){lang="html"} 组件，它在服务器端获取帖子数据，并通过插槽暴露数据，完全支持样式自定义。所有数据都通过你的服务器代理 — 无需客户端对 Bluesky 进行 API 调用。

::script-stats
::

::script-types
::

## 安装配置

要使用 Bluesky 嵌入组件，必须在你的 `nuxt.config` 中启用它：

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      blueskyEmbed: true,
    },
  },
})
```

这会注册所需的服务器 API 路由（`/_scripts/embed/bluesky` 和 `/_scripts/embed/bluesky-image`），用于处理帖子数据获取及图片代理。

## [`<ScriptBlueskyEmbed>`{lang="html"}](/scripts/bluesky-embed){lang="html"}

[`<ScriptBlueskyEmbed>`{lang="html"}](/scripts/bluesky-embed){lang="html"} 组件是一个无头组件，特点：
- 通过 Bluesky 公共 API（AT Protocol）在服务器端获取帖子数据
- 所有图片通过你的服务器代理，保护隐私
- 将富文本中的特性（链接、提及、标签）转换为 HTML
- 通过作用域插槽暴露帖子数据，支持自定义渲染
- 缓存响应 10 分钟
- 遵守作者选择退出机制（`!no-unauthenticated` 标签）

### 示例

::code-group

```vue [基本用法]
<template>
  <ScriptBlueskyEmbed post-url="https://bsky.app/profile/bsky.app/post/3mgnwwvj3u22a">
    <template #default="{ displayName, handle, text, datetime, likesFormatted }">
      <div class="border rounded-lg p-4 max-w-md">
        <p class="font-bold">
          {{ displayName }} (@{{ handle }})
        </p>
        <p>{{ text }}</p>
        <p class="text-gray-500 text-sm">
          {{ datetime }} - {{ likesFormatted }} 点赞
        </p>
      </div>
    </template>
  </ScriptBlueskyEmbed>
</template>
```

```vue [Bluesky 卡片（Tailwind）]
<template>
  <ScriptBlueskyEmbed post-url="https://bsky.app/profile/bsky.app/post/3mgnwwvj3u22a">
    <template #default="{ displayName, handle, avatar, richText, datetime, likes, likesFormatted, reposts, repostsFormatted, replies, images, externalEmbed, postUrl, authorUrl }">
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
          <div class="text-gray-900 dark:text-white whitespace-pre-wrap break-words leading-[22px] [&_a]:text-[#1185fe] [&_a]:no-underline [&_a:hover]:underline" v-html="richText" />
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
            <img v-if="externalEmbed.thumb" :src="externalEmbed.thumb" class="w-full aspect-video object-cover">
            <div class="px-3 py-2">
              <div class="font-semibold text-gray-900 dark:text-white text-[15px] leading-5 line-clamp-2">{{ externalEmbed.title }}</div>
              <div class="text-gray-500 dark:text-[#abb8c9] text-[13px] leading-[17px] line-clamp-2 mt-0.5">{{ externalEmbed.description }}</div>
              <div class="flex items-center gap-1 mt-1 text-[11px] text-gray-400 dark:text-[#abb8c9]">
                <svg fill="none" viewBox="0 0 24 24" width="12" height="12"><path fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" d="M4.4 9.493C4.14 10.28 4 11.124 4 12a8 8 0 1 0 10.899-7.459l-.953 3.81a1 1 0 0 1-.726.727l-3.444.866-.772 1.533a1 1 0 0 1-1.493.35L4.4 9.493Zm.883-1.84L7.756 9.51l.44-.874a1 1 0 0 1 .649-.52l3.306-.832.807-3.227a7.993 7.993 0 0 0-7.676 3.597ZM2 12C2 6.477 6.477 2 12 2s10 4.477 10 10-4.477 10-10 10S2 17.523 2 12Zm8.43.162a1 1 0 0 1 .77-.29l1.89.121a1 1 0 0 1 .494.168l2.869 1.928a1 1 0 0 1 .336 1.277l-.973 1.946a1 1 0 0 1-.894.553h-2.92a1 1 0 0 1-.831-.445L9.225 14.5a1 1 0 0 1 .126-1.262l1.08-1.076Z" /></svg>
                {{ new URL(externalEmbed.uri).hostname }}
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

        <!-- 操作按钮 -->
        <div class="flex items-center justify-between px-4 py-2 border-t border-gray-200 dark:border-[#2c3a4e]">
          <div class="flex items-center gap-8">
            <!-- 回复 -->
            <button class="text-gray-400 dark:text-[#6f839f] hover:text-[#1185fe] transition-colors" aria-label="回复">
              <svg fill="none" width="20" viewBox="0 0 24 24" height="20"><path fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" d="M20 7a2 2 0 0 0-2-2H6a2 2 0 0 0-2 2v8a2 2 0 0 0 2 2h2a1 1 0 0 1 1 1v1.918l3.375-2.7a1 1 0 0 1 .625-.218h5a2 2 0 0 0 2-2V7Zm2 8a4 4 0 0 1-4 4h-4.648l-4.727 3.781A1.001 1.001 0 0 1 7 22v-3H6a4 4 0 0 1-4-4V7a4 4 0 0 1 4-4h12a4 4 0 0 1 4 4v8Z" /></svg>
            </button>
            <!-- 转发 -->
            <button class="text-gray-400 dark:text-[#6f839f] hover:text-green-500 transition-colors" aria-label="转发">
              <svg fill="none" width="20" viewBox="0 0 24 24" height="20"><path fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" d="M17.957 2.293a1 1 0 1 0-1.414 1.414L17.836 5H6a3 3 0 0 0-3 3v3a1 1 0 1 0 2 0V8a1 1 0 0 1 1-1h11.836l-1.293 1.293a1 1 0 0 0 1.414 1.414l2.47-2.47a1.75 1.75 0 0 0 0-2.474l-2.47-2.47ZM20 12a1 1 0 0 1 1 1v3a3 3 0 0 1-3 3H6.164l1.293 1.293a1 1 0 1 1-1.414 1.414l-2.47-2.47a1.75 1.75 0 0 1 0-2.474l2.47-2.47a1 1 0 0 1 1.414 1.414L6.164 17H18a1 1 0 0 0 1-1v-3a1 1 0 0 1 1-1Z" /></svg>
            </button>
            <!-- 点赞 -->
            <button class="text-gray-400 dark:text-[#6f839f] hover:text-pink-500 transition-colors" aria-label="点赞">
              <svg fill="none" width="20" viewBox="0 0 24 24" height="20"><path fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" d="M16.734 5.091c-1.238-.276-2.708.047-4.022 1.38a1 1 0 0 1-1.424 0C9.974 5.137 8.504 4.814 7.266 5.09c-1.263.282-2.379 1.206-2.92 2.556C3.33 10.18 4.252 14.84 12 19.348c7.747-4.508 8.67-9.168 7.654-11.7-.541-1.351-1.657-2.275-2.92-2.557Zm4.777 1.812c1.604 4-.494 9.69-9.022 14.47a1 1 0 0 1-.978 0C2.983 16.592.885 10.902 2.49 6.902c.779-1.942 2.414-3.334 4.342-3.764 1.697-.378 3.552.003 5.169 1.286 1.617-1.283 3.472-1.664 5.17-1.286 1.927.43 3.562 1.822 4.34 3.764Z" /></svg>
            </button>
          </div>
          <div class="flex items-center gap-4">
            <!-- 分享 -->
            <a :href="postUrl" target="_blank" rel="noopener noreferrer" class="text-gray-400 dark:text-[#6f839f] hover:text-[#1185fe] transition-colors" aria-label="在 Bluesky 打开">
              <svg fill="none" width="20" viewBox="0 0 24 24" height="20"><path fill="currentColor" fill-rule="evenodd" clip-rule="evenodd" d="M11.839 4.744c0-1.488 1.724-2.277 2.846-1.364l.107.094 7.66 7.256.128.134c.558.652.558 1.62 0 2.272l-.128.135-7.66 7.255c-1.115 1.057-2.953.267-2.953-1.27v-2.748c-3.503.055-5.417.41-6.592.97-.997.474-1.525 1.122-2.084 2.14l-.243.46c-.558 1.088-2.09.583-2.08-.515l.015-.748c.111-3.68.777-6.5 2.546-8.415 1.83-1.98 4.63-2.771 8.438-2.884V4.744Z" /></svg>
            </a>
          </div>
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
          {{ datetime }} - {{ likesFormatted }} 点赞
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
  avatar: string // 代理后的 URL
  avatarOriginal: string // 原始 Bluesky CDN URL
  isVerified: boolean
  // 帖子内容
  text: string // 纯文本
  richText: string // 含链接、提及和标签的 HTML
  langs?: string[] // 语言代码
  // 格式化数值
  datetime: string // "12:47 PM · 2024年2月5日"
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
| `loading` | 数据获取中显示 |
| `error` | 获取失败时显示，接收 `{ error }` |

## 工作原理

1. **服务器端获取**：服务器在 SSR 期间从 `public.api.bsky.app`（AT 协议）获取帖子数据
2. **解析用户名**：服务器将用户名解析为 DID，以便可靠地查找帖子
3. **图片代理**：服务器重写所有图片，通过 `/_scripts/embed/bluesky-image` 代理
4. **富文本**：组件将 Bluesky 的 facets（链接、提及、标签）转换为 HTML
5. **缓存**：服务器缓存响应 10 分钟
6. **无客户端 API 调用**：用户浏览器永远不会直接联系 Bluesky

## 隐私优势

- 不加载第三方 JavaScript
- Bluesky 不设置 Cookie
- 浏览器与 Bluesky 无直接通信
- 用户 IP 地址不与 Bluesky 共享
- 所有内容均从您的域名提供

## 作者选择退出

该组件尊重 Bluesky 的 `!no-unauthenticated` 标签。如果帖子作者选择不允许外部嵌入，API 会返回 403 错误，组件显示错误插槽。
