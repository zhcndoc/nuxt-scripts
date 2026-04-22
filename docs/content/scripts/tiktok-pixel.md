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

::script-types
::
