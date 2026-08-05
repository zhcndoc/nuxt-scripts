---
title: Instagram 嵌入
description: 无需浏览器直接请求 Instagram 的服务端渲染 Instagram 嵌入内容。
links:
  - label: ScriptInstagramEmbed
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptInstagramEmbed.vue
    size: xs
---

[Instagram](https://instagram.com) 托管照片和视频帖子。

[`<ScriptInstagramEmbed>`{lang="html"}](/scripts/instagram-embed){lang="html"} 通过您的服务器获取嵌入 HTML，并代理受支持的媒体和静态资源。在客户端导航后更改帖子 URL 或标题设置，会重新获取您的 Nuxt 端点。

::script-stats
::

::script-docs{embed}
::

这会注册所需的服务器 API 路由（`/_scripts/embed/instagram`、`/_scripts/embed/instagram-image` 和 `/_scripts/embed/instagram-asset`），用于处理嵌入 HTML 的获取以及图像/资源的代理。

## [`<ScriptInstagramEmbed>`{lang="html"}](/scripts/instagram-embed){lang="html"}

### 演示

::code-group

:instagram-embed-demo{label="输出"}

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
| --- | --- | --- | --- |
| `postUrl` | `string` | 必填 | Instagram 帖子链接（例如 `https://www.instagram.com/p/ABC123/`） |
| `captions` | `boolean` | `true` | 是否在嵌入中包含字幕 |
| `apiEndpoint` | `string` | `/api/_scripts/instagram-embed` | 用于获取嵌入 HTML 的自定义 API 端点 |
| `rootAttrs` | `HTMLAttributes` | `{}` | 根元素属性 |

### 插槽 Props

默认插槽接收：

```ts
interface SlotProps {
  html: string // 已处理的嵌入 HTML，已移除脚本并注入作用域 CSS
  shortcode: string // 帖子短代码（例如 "C3Sk6d2MTjI"）
  postUrl: string // 原始帖子 URL
}
```

重写器会移除 `script`、`noscript` 和原始 `style` 元素。它会获取链接的 Instagram 样式表，为其中的选择器添加作用域，并将生成的 CSS 注入返回的 HTML 中。它不会执行通用的 HTML 清理。默认来源仅限于 Instagram。如果你通过自己的渲染器传递 `html`，或引入自定义端点，请根据应用的内容策略对其进行清理，然后再使用 `v-html`。

### 具名插槽

| 插槽 | 说明 |
| --- | --- |
| `default` | 主要内容，接收 `{ html, shortcode, postUrl }`。默认渲染 HTML。 |
| `loading` | 获取嵌入 HTML 时显示 |
| `error` | 嵌入获取失败时显示，接收 `{ error }` |

## 支持的 URL 格式

- 帖子：`https://www.instagram.com/p/ABC123/`
- Reels：`https://www.instagram.com/reel/ABC123/`
- TV：`https://www.instagram.com/tv/ABC123/`

## 工作原理

1. **服务端获取**：Nuxt 从 `{postUrl}/embed/` 获取 Instagram 嵌入 HTML
2. **资源代理**：来自 Instagram 媒体主机的图片以及来自 `static.cdninstagram.com` 的资源会被重写为通过你的服务器进行代理
3. **移除脚本**：Nuxt 移除 Instagram 的 `embed.js`（静态渲染不需要）
4. **缓存**：Nuxt 在服务器级别缓存响应 10 分钟

## 浏览器隐私

渲染后的嵌入内容不会加载 Instagram JavaScript，不会转发 Meta 的 `Set-Cookie` 响应，并会将图片和资源请求发送到你的 Nuxt 服务器。Instagram 看到的是服务器的连接，而不是访客的连接。如果访客选择打开嵌入内容中的链接，仍然可以前往 Instagram。

## 限制

- 仅支持单图帖子（画廊仅显示第一张图片）
- 视频显示为静态海报图片
- 某些互动功能不可用（点赞、评论）
- 图片端点会拒绝重定向。静态资源端点目前会跟随重定向，而不会重新验证目标主机。

::script-types
::
