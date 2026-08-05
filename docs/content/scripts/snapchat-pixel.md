---
title: Snapchat Pixel
description: 加载 Snapchat Pixel，并选择自动或手动触发页面浏览事件。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/snapchat-pixel.ts
  size: xs
---

[Snapchat Pixel](https://businesshelp.snapchat.com/s/article/snap-pixel-about){:target="_blank"} 会将浏览器事件报告给 Snapchat Ads，用于转化衡量。

使用 [`useScriptSnapchatPixel()`{lang="ts"}](/scripts/snapchat-pixel){lang="ts"} 加载 Pixel 并访问其 `snaptr` API。

::script-stats
::

::script-docs
::

## 页面浏览量

此集成默认不会发送 `PAGE_VIEW`。请显式启用初始化事件。Snapchat 的[官方页面浏览量代码片段](https://businesshelp.snapchat.com/articles/en_US/Knowledge/pixel-bigcommerce)使用相同的 `snaptr('track', 'PAGE_VIEW')`{lang="ts"} 命令：

```ts
const { proxy } = useScriptSnapchatPixel({
  id: 'YOUR_PIXEL_ID',
  trackPageView: true,
})

// 当您的跟踪政策要求时，稍后发送页面浏览量。
proxy.snaptr('track', 'PAGE_VIEW')
```

::script-types
::
