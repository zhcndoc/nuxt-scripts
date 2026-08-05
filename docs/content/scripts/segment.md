---
title: Segment
description: 加载 Segment Analytics.js 并将页面、事件、身份和群组调用加入队列。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/segment.ts
  size: xs
---

[Segment](https://www.twilio.com/en-us/segment) 将您网站中的事件发送到分析、营销和数据仓库目标。

使用 [`useScriptSegment()`{lang="ts"}](/scripts/segment){lang="ts"} 加载 Analytics.js 并访问其跟踪方法。Segment 的 [Analytics.js 快速入门指南](https://www.twilio.com/docs/segment/connections/sources/catalog/libraries/website/javascript/quickstart) 介绍了写入密钥，以及该库提供的 `page`、`track` 和 `identify` 调用。

客户端初始化器会在 Analytics.js 加载前将一个 `page()`{lang="ts"} 调用加入队列。除非您有意将其计数两次，否则请避免为初始路由发送另一个页面调用。

::script-stats
::

::script-docs
::

::script-types
::
