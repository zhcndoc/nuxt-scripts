---
title: TikTok Pixel
description: 在你的 Nuxt 应用中使用 TikTok Pixel。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/tiktok-pixel.ts
  size: xs
---

[TikTok Pixel](https://ads.tiktok.com/help/article/tiktok-pixel) 让你能够衡量、优化并构建 TikTok 广告活动的受众。

Nuxt Scripts 提供了注册脚本组合函数 `useScriptTikTokPixel`，轻松地在你的 Nuxt 应用中集成 TikTok Pixel。

### Nuxt 配置设置

在 Nuxt 应用中全局加载 TikTok Pixel 最简单的方式是使用 Nuxt 配置。你也可以直接使用 [useScriptTikTokPixel](#usescripttiktokpixel) 组合函数。

如果你不打算发送自定义事件，可以使用 [环境配置覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 在开发环境中禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      tiktokPixel: {
        id: 'YOUR_PIXEL_ID'
      }
    }
  }
})
```

```ts [仅生产环境]
export default defineNuxtConfig({
  $production: {
    scripts: {
      registry: {
        tiktokPixel: {
          id: 'YOUR_PIXEL_ID'
        }
      }
    }
  }
})
```

::

#### 使用环境变量

如果你更倾向于使用环境变量配置你的 id。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      tiktokPixel: true,
    }
  },
  // 需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        tiktokPixel: {
          id: '', // 对应 NUXT_PUBLIC_SCRIPTS_TIKTOK_PIXEL_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_TIKTOK_PIXEL_ID=<YOUR_ID>
```

## useScriptTikTokPixel

`useScriptTikTokPixel` 组合函数让你能够细粒度地控制 TikTok Pixel 在你网站上的加载时机和方式。

```ts
const { proxy } = useScriptTikTokPixel({
  id: 'YOUR_PIXEL_ID'
})

// 跟踪事件
proxy.ttq('track', 'ViewContent', {
  content_id: '123',
  content_name: '产品名称',
  value: 99.99,
  currency: 'USD'
})
```

请参考 [注册脚本](/docs/guides/registry-scripts) 指南了解更多高级用法。

### TikTokPixelApi

```ts
export interface TikTokPixelApi {
  ttq: TtqFns & {
    push: TtqFns
    loaded: boolean
    queue: any[]
  }
}

type TtqFns =
  & ((cmd: 'track', event: StandardEvents | string, properties?: EventProperties) => void)
  & ((cmd: 'page') => void)
  & ((cmd: 'identify', properties: IdentifyProperties) => void)
  & ((cmd: string, ...args: any[]) => void)

type StandardEvents =
  | 'ViewContent' | 'ClickButton' | 'Search' | 'AddToWishlist'
  | 'AddToCart' | 'InitiateCheckout' | 'AddPaymentInfo' | 'CompletePayment'
  | 'PlaceAnOrder' | 'Contact' | 'Download' | 'SubmitForm'
  | 'CompleteRegistration' | 'Subscribe'
```

### 配置 Schema

首次设置脚本时必须提供选项。

```ts
export const TikTokPixelOptions = object({
  id: string(),
  trackPageView: optional(boolean()), // 默认: true
})
```

## 示例

使用 TikTok Pixel 跟踪购买事件。

::code-group

```vue [PurchaseButton.vue]
<script setup lang="ts">
const { proxy } = useScriptTikTokPixel()

function trackPurchase() {
  proxy.ttq('track', 'CompletePayment', {
    content_id: 'product-123',
    content_name: '超棒的产品',
    value: 49.99,
    currency: 'USD'
  })
}
</script>

<template>
  <button @click="trackPurchase">
    完成购买
  </button>
</template>
```

::

## 用户识别

你可以识别用户进行高级匹配：

```ts
const { proxy } = useScriptTikTokPixel()

proxy.ttq('identify', {
  email: 'user@example.com',
  phone_number: '+1234567890'
})
```

## 禁用自动页面浏览跟踪

默认情况下，TikTok Pixel 会自动追踪页面浏览。若要禁用：

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