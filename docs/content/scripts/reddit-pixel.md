---
title: Reddit Pixel
description: 加载 Reddit Pixel，并通过其类型化 API 发送页面访问或转化事件。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/reddit-pixel.ts
  size: xs
---

[Reddit Pixel](https://business.reddithelp.com/s/article/reddit-pixel) 可帮助你跟踪转化，并为 Reddit 广告活动构建受众群体。

使用 [`useScriptRedditPixel()`{lang="ts"}](/scripts/reddit-pixel){lang="ts"} 加载 Reddit Pixel 并访问其 `rdt` API。

::script-stats  
::

::script-docs  
::

## 初始页面访问

客户端初始化器在排队执行 `init` 后，会将 `rdt('track', 'PageVisit')`{lang="ts"} 排队执行一次。Nuxt 路由发生变化时，它不会再次发送页面访问事件。如果每次客户端导航都应计为一次 [`PageVisit`](https://business.reddithelp.com/articles/Knowledge/supported-conversion-events)，请添加自己的路由跟踪。

::script-types
::
