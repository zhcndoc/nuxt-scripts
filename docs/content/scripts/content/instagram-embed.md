---
title: Instagram 嵌入
description: 服务器端渲染的 Instagram 嵌入，无需客户端 API 调用。
links:
  - label: ScriptInstagramEmbed
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptInstagramEmbed.vue
    size: xs
---

[Instagram](https://instagram.com) 是一个照片和视频分享的社交媒体平台。

Nuxt Scripts 提供了一个 `ScriptInstagramEmbed` 组件，它在服务器端获取 Instagram 的嵌入 HTML，并通过你的服务器代理所有资源 —— 无需客户端调用 Instagram API。

## ScriptInstagramEmbed

`ScriptInstagramEmbed` 组件特点：
- 服务器端获取官方 Instagram 嵌入 HTML
- 重写所有图片和资源 URL，通过你的服务器代理
- 移除 Instagram 的 embed.js 脚本（不需要）
- 缓存响应 10 分钟

### 示例

::code-group

```vue [基础用法]
<template>
  <ScriptInstagramEmbed
    post-url="https://www.instagram.com/p/C3Sk6d2MTjI/"
    :captions="true"
  />
</template>
```

```vue [自定义加载/错误状态]
<template>
  <ScriptInstagramEmbed
    post-url="https://www.instagram.com/p/C3Sk6d2MTjI/"
    :captions="true"
  >
    <template #loading>
      <div class="animate-pulse bg-gray-100 rounded-lg p-4 aspect-square max-w-md">
        正在加载 Instagram 帖子...
      </div>
    </template>

    <template #error>
      <div class="bg-red-50 border border-red-200 rounded-lg p-4 max-w-md">
        加载 Instagram 帖子失败
      </div>
    </template>
  </ScriptInstagramEmbed>
</template>
```

```vue [自定义渲染]
<template>
  <ScriptInstagramEmbed post-url="https://www.instagram.com/p/C3Sk6d2MTjI/">
    <template #default="{ html, shortcode, postUrl }">
      <div class="instagram-wrapper">
        <a :href="postUrl" target="_blank" class="text-sm text-gray-500 mb-2 block">
          在 Instagram 查看 ({{ shortcode }})
        </a>
        <!-- eslint-disable-next-line vue/no-v-html -->
        <div v-html="html" />
      </div>
    </template>
  </ScriptInstagramEmbed>
</template>
```

::

### Props

`ScriptInstagramEmbed` 组件接受以下属性：

| 属性 | 类型 | 默认值 | 说明 |
|------|------|---------|-------------|
| `postUrl` | `string` | 必填 | Instagram 帖子链接 (例如 `https://www.instagram.com/p/ABC123/`) |
| `captions` | `boolean` | `true` | 是否在嵌入中包含字幕 |
| `apiEndpoint` | `string` | `/api/_scripts/instagram-embed` | 用于获取嵌入 HTML 的自定义 API 端点 |
| `rootAttrs` | `HTMLAttributes` | `{}` | 根元素属性 |

### 插槽 Props

默认插槽接收：

```ts
interface SlotProps {
  html: string      // 处理后的嵌入 HTML
  shortcode: string // 帖子短码 (例如 "C3Sk6d2MTjI")
  postUrl: string   // 原始帖子链接
}
```

### 命名插槽

| 插槽 | 说明 |
|------|-------------|
| `default` | 主要内容，接收 `{ html, shortcode, postUrl }`。默认渲染 HTML。 |
| `loading` | 获取嵌入 HTML 时显示 |
| `error` | 嵌入获取失败时显示，接收 `{ error }` |

## 支持的 URL 格式

- 帖子: `https://www.instagram.com/p/ABC123/`
- Reels: `https://www.instagram.com/reel/ABC123/`
- TV: `https://www.instagram.com/tv/ABC123/`

## 工作原理

1. **服务器端抓取**：从 `{postUrl}/embed/` 获取 Instagram 嵌入 HTML
2. **资源代理**：将来自 `scontent.cdninstagram.com` 的所有图片和来自 `static.cdninstagram.com` 的资源重写为通过你的服务器代理
3. **脚本移除**：移除 Instagram 的 `embed.js` （静态渲染不需要）
4. **缓存**：服务器级别缓存响应 10 分钟

该方案启发自 [Cloudflare Zaraz 的嵌入实现](https://blog.cloudflare.com/zaraz-supports-server-side-rendering-of-embeds/)。

## 隐私优势

- 不加载第三方 JavaScript
- Instagram/Meta 不设置任何 Cookies
- 浏览器不会直接与 Instagram 通信
- 不会向 Instagram 共享用户 IP 地址
- 所有内容均由你的域名提供

## 限制

- 仅支持单图帖子（图集只显示第一张图片）
- 视频以静态封面图展示
- 一些交互功能不可用（点赞、评论）