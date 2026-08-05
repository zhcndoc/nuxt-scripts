---
title: X Pixel
description: 通过浏览器和服务器事件去重发送 X Ads 转化事件。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/x-pixel.ts
  size: xs
---

[X Pixel](https://business.x.com/en/help/campaign-measurement-and-analytics/conversion-tracking-for-websites) 可记录网站转化，并为 X Ads 广告系列构建受众群体。

使用 [`useScriptXPixel()`{lang="ts"}](/scripts/x-pixel){lang="ts"} 加载像素并访问其 `twq` API。

::script-stats  
::

::script-docs  
::

## 浏览器和服务器事件去重

当你从浏览器和服务器集成发送同一个转化时，请为两个事件使用相同的唯一 `conversion_id`。X 会使用该值对这两个事件进行去重，具体说明请参阅其[转化跟踪指南](https://business.x.com/en/help/campaign-measurement-and-analytics/conversion-tracking-for-websites)：

```ts
const { proxy } = useScriptXPixel({ id: 'YOUR_PIXEL_ID' })

proxy.twq('event', 'YOUR_EVENT_ID', {
  conversion_id: order.id,
  value: order.total,
  currency: 'USD',
  contents: order.items.map(item => ({
    content_id: item.id,
    num_items: item.quantity,
  })),
})
```

当前 Nuxt Scripts 类型要求每个事件都包含 `contents`。X 将其列为事件参数，而不是所有事件的通用要求；只有其动态产品广告说明要求所有事件都包含该参数。目前请暂时包含它；如果你的事件没有商品数据，也可以通过本地包装器进行收窄。

::script-types
::
