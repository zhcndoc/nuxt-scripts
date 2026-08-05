---
title: TikTok Pixel
description: 在获得用户同意的情况下，通过去重和高级匹配跟踪 TikTok 转化。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/tiktok-pixel.ts
  size: xs
---

[TikTok Pixel](https://ads.tiktok.com/help/article/tiktok-pixel) 会将浏览器事件报告给 TikTok Ads，用于转化衡量和受众分析。

使用 [`useScriptTikTokPixel()`{lang="ts"}](/scripts/tiktok-pixel){lang="ts"} 加载像素并访问其 `ttq` API。

::script-stats
::

::script-docs
::

## 禁用自动页面浏览

默认情况下，TikTok Pixel 会在初始化期间跟踪页面浏览。可在组合式函数调用中禁用此功能：

```ts
useScriptTikTokPixel({
  id: 'YOUR_PIXEL_ID',
  trackPageView: false,
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

完整行为请参阅 [TikTok Cookie 同意文档](https://business-api.tiktok.com/portal/docs?id=1739585600931842)。

初始同意命令会在像素初始化之前排队，但不会延迟 SDK 请求。如果你的政策要求在用户选择同意之前不得向 TikTok 发起请求，请为脚本本身使用[同意触发器](/docs/guides/consent#binary-load-gate)。

## 数据驻留端点

设置 `region: 'us'`，即可从 `analytics.us.tiktok.com` 而非全局主机加载 Pixel SDK：

```ts
useScriptTikTokPixel({
  id: 'YOUR_PIXEL_ID',
  region: 'us',
})
```

此选项仅用于选择 SDK 主机。它本身并不能证明您的其余跟踪设置符合数据驻留或隐私要求。

## 服务端事件去重

对于 Pixel + Events API（CAPI）模式，请在浏览器端和服务端都传递相同的 `event_id`，这样 TikTok 就会对这对事件进行去重：

```vue
<script setup lang="ts">
const { proxy } = useScriptTikTokPixel({ id: 'YOUR_PIXEL_ID' })

async function checkout(order: { id: string, total: number }) {
  const eventId = crypto.randomUUID()

  proxy.ttq.track('Purchase', { value: order.total, currency: 'USD', order_id: order.id }, { event_id: eventId })

  await $fetch('/api/tiktok/event', {
    method: 'POST',
    body: { event: 'Purchase', event_id: eventId, order_id: order.id, value: order.total },
  })
}
</script>
```

有关完整规则，请参阅 [TikTok 的事件去重指南](https://ads.tiktok.com/help/article/event-deduplication?lang=en)。

## 测试浏览器事件

TikTok 的[浏览器 Pixel 测试指南](https://ads.us.tiktok.com/help/article/test-tiktok-pixel-events-video-walkthrough?lang=en)使用 Events Manager 中的 Test Events 选项卡：输入网站 URL，通过生成的测试流程打开该网站，然后执行你想要检查的操作。

当前的 Nuxt Scripts 类型还接受第四个 `track` 参数中的 `test_event_code`，并将其转发给 SDK。TikTok 为服务端 Events API 记录了该字段，但并未将其用于浏览器 `ttq.track` 签名，因此不要依赖它来进行浏览器测试。

## 高级匹配

Nuxt Scripts 要求每个识别字段（`email`、`phone_number`、`external_id`、`first_name`、`last_name`、`city`、`state`、`country`、`zip_code`）都使用 64 个字符的 SHA-256 十六进制摘要。TikTok 的[高级匹配指南](https://ads.tiktok.com/help/article/advanced-matching-web?lang=en)介绍了规范化和哈希处理。在开发环境中，当某个值看起来不像经过哈希处理时，该组合式函数会发出警告。

```ts
async function sha256(value: string) {
  const input = new TextEncoder().encode(value)
  const digest = await crypto.subtle.digest('SHA-256', input)
  return Array.from(new Uint8Array(digest), byte => byte.toString(16).padStart(2, '0')).join('')
}

const { proxy } = useScriptTikTokPixel({ id: 'YOUR_PIXEL_ID' })
proxy.ttq.identify({
  email: await sha256('user@example.com'.trim().toLowerCase()),
  phone_number: await sha256('+15551234567'),
})
```

[`crypto.subtle.digest()`{lang="ts"}](https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest) 在浏览器中仅可用于安全上下文。本地主机在开发环境中会被视为安全环境；请通过 HTTPS 部署此示例。

::script-types
::
