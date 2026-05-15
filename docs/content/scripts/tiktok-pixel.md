---
title: TikTok 像素
description: 在你的 Nuxt 应用中使用 TikTok 像素。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/tiktok-pixel.ts
  size: xs
---

[TikTok 像素](https://ads.tiktok.com/help/article/tiktok-pixel) 让你能够衡量、优化并为你的 TikTok 广告活动构建受众。

Nuxt Scripts 提供了一个注册表脚本组合式函数 [`useScriptTikTokPixel()`{lang="ts"}](/scripts/tiktok-pixel){lang="ts"}，方便你在 Nuxt 应用中集成 TikTok 像素。

::script-stats
::

::script-docs
::

## 识别用户

你可以识别用户以进行高级匹配：

```ts
const { proxy } = useScriptTikTokPixel()

proxy.ttq('identify', {
  email: 'user@example.com',
  phone_number: '+1234567890'
})
```

## 禁用自动页面浏览追踪

默认情况下，TikTok 像素会自动追踪页面浏览。若要禁用：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      tiktokPixel: {
        id: 'YOUR_PIXEL_ID',
        trackPageView: false
      }
    }
  }
})
```

## 同意模式

TikTok 像素提供了一个三态同意 API：授权、撤销，或保持（延后决定）。使用 `defaultConsent` 设置初始状态，并在运行时调用 `consent.grant()`{lang="ts"} / `consent.revoke()`{lang="ts"} / `consent.hold()`{lang="ts"}：

```vue
<script setup lang="ts">
const { consent } = useScriptTikTokPixel({
  id: 'YOUR_PIXEL_ID',
  defaultConsent: 'hold', // 'granted' | 'denied' | 'hold'
})

function acceptAds() {
  consent.grant()
}
function rejectAds() {
  consent.revoke()
}
</script>
```

有关完整行为，请参阅 [TikTok cookie 同意文档](https://business-api.tiktok.com/portal/docs?id=1739585600931842)。

## 数据驻留区域

有美国数据驻留要求的企业可以通过设置 `region: 'us'`（默认值为 `'global'`）将 Pixel SDK 路由至 `analytics.us.tiktok.com`：

```ts
useScriptTikTokPixel({
  id: 'YOUR_PIXEL_ID',
  region: 'us',
})
```

## 服务端事件去重

对于 Pixel + Events API（CAPI）模式，请在浏览器端和服务端都传递相同的 `event_id`，这样 TikTok 就会对这对事件进行去重：

```vue
<script setup lang="ts">
const { proxy } = useScriptTikTokPixel({ id: 'YOUR_PIXEL_ID' })

async function checkout(order: { id: string, total: number }) {
  const eventId = crypto.randomUUID()

  proxy.ttq('track', 'Purchase', { value: order.total, currency: 'USD', order_id: order.id }, { event_id: eventId })

  await $fetch('/api/tiktok/event', {
    method: 'POST',
    body: { event: 'Purchase', event_id: eventId, order_id: order.id, value: order.total },
  })
}
</script>
```

有关完整规则，请参阅 [TikTok 的事件去重指南](https://ads.tiktok.com/help/article/event-deduplication?lang=en)。

## 测试事件沙盒

在第 4 个 `track` 参数中设置 `test_event_code`，即可将事件路由到 TikTok 的测试事件面板，而不会影响生产环境报告：

```ts
proxy.ttq('track', 'Purchase', { value: 99 }, { test_event_code: 'TEST12345' })
```

## 高级匹配

TikTok 要求识别字段（`email`、`phone_number`、`external_id`、`first_name`、`last_name`、`city`、`state`、`country`、`zip_code`）必须是经过 SHA-256 哈希处理的小写值。TikTok 会静默丢弃原始值；在开发环境中，如果 Nuxt Scripts 检测到未哈希的值，会记录警告：

```ts
import { sha256 } from 'ohash'

const { proxy } = useScriptTikTokPixel({ id: 'YOUR_PIXEL_ID' })
proxy.ttq('identify', {
  email: sha256('user@example.com'.trim().toLowerCase()),
  phone_number: sha256('+15551234567'),
})
```

::script-types
::
