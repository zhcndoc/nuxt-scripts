---
title: Snapchat 像素
description: 在你的 Nuxt 应用中使用 Snapchat 像素。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/snapchat-pixel.ts
  size: xs
---

[Snapchat 像素](https://businesshelp.snapchat.com/s/article/snap-pixel-about){:target="_blank"} 让你能够衡量 Snapchat 广告活动的跨设备影响。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptSnapchatPixel`，方便你在 Nuxt 应用中集成 Snapchat 像素。

### Nuxt 配置设置

在 Nuxt 应用中全局加载 Snapchat 像素的最简单方式是使用 Nuxt 配置。或者，你也可以直接使用[useScriptSnapchatPixel](#useScriptSnapchatPixel)组合式函数。

如果你不打算发送自定义事件，可以使用[环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides)来在开发环境禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      snapchatPixel: {
        id: 'YOUR_ID'
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
        snapchatPixel: {
          id: 'YOUR_ID',
        }
      }
    }
  }
})
```

::

#### 使用环境变量配置

如果你更喜欢通过环境变量来配置你的 ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      snapchatPixel: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        snapchatPixel: {
          id: '', // NUXT_PUBLIC_SCRIPTS_SNAPCHAT_PIXEL_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_SNAPCHAT_PIXEL_ID=<YOUR_ID>
```

## useScriptSnapchatPixel

`useScriptSnapchatPixel` 组合式函数允许你精细控制 Snapchat 像素在你的网站上何时以及如何加载。

```ts
const { proxy } = useScriptSnapchatPixel({
  id: 'YOUR_ID',
  user_email: 'USER_EMAIL'
})
// 示例
proxy.snaptr('track', 'PURCHASE', {
  currency: 'USD',
  price: 120.10,
  transaction_id: '11111'
})
```

请参阅[注册脚本](/docs/guides/registry-scripts)指南，了解更多高级用法。

### SnapchatPixelApi

```ts
export interface SnapPixelApi {
  snaptr: SnapTrFns & {
    push: SnapTrFns
    loaded: boolean
    version: string
    queue: any[]
  }
  _snaptr: SnapPixelApi['snaptr']
  handleRequest?: SnapTrFns
}
type StandardEvents = 'PAGE_VIEW' | 'VIEW_CONTENT' | 'ADD_CART' | 'SIGN_UP' | 'SAVE' | 'START_CHECKOUT' | 'APP_OPEN' | 'ADD_BILLING' | 'SEARCH' | 'SUBSCRIBE' | 'AD_CLICK' | 'AD_VIEW' | 'COMPLETE_TUTORIAL' | 'LEVEL_COMPLETE' | 'INVITE' | 'LOGIN' | 'SHARE' | 'RESERVE' | 'ACHIEVEMENT_UNLOCKED' | 'ADD_TO_WISHLIST' | 'SPENT_CREDITS' | 'RATE' | 'START_TRIAL' | 'LIST_VIEW'
type SnapTrFns =
  ((event: 'track', eventName: StandardEvents | '', data?: EventObjectProperties) => void) &
  ((event: 'init', id: string, data?: Record<string, any>) => void) &
  ((event: 'init', id: string, data?: InitObjectProperties) => void) &
  ((event: string, ...params: any[]) => void)
interface EventObjectProperties {
  price?: number
  client_dedup_id?: string
  currency?: string
  transaction_id?: string
  item_ids?: string[]
  item_category?: string
  description?: string
  search_string?: string
  number_items?: number
  payment_info_available?: 0 | 1
  sign_up_method?: string
  success?: 0 | 1
  brands?: string[]
  delivery_method?: 'in_store' | 'curbside' | 'delivery'
  customer_status?: 'new' | 'returning' | 'reactivated'
  event_tag?: string
  [key: string]: any
}
interface InitObjectProperties {
  user_email?: string
  ip_address?: string
  user_phone_number?: string
  user_hashed_email?: string
  user_hashed_phone_number?: string
  firstname?: string
  lastname?: string
  geo_city?: string
  geo_region?: string
  geo_postal_code?: string
  geo_country?: string
  age?: string
}
```

### 配置 Schema

首次设置脚本时必须提供选项。

```ts
export const SnapTrPixelOptions = object({
  id: string(),
  trackPageView: optional(boolean()),
  user_email: optional(string()),
  ip_address: optional(string()),
  user_phone_number: optional(string()),
  user_hashed_email: optional(string()),
  user_hashed_phone_number: optional(string()),
  firstname: optional(string()),
  lastname: optional(string()),
  geo_city: optional(string()),
  geo_region: optional(string()),
  geo_postal_code: optional(string()),
  geo_country: optional(string()),
  age: optional(string()),
})
```

## 示例

仅在生产环境中使用 Snapchat 像素，并通过 `snaptr` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptSnapchatPixel()

// 在开发环境和 SSR 中为 noop
// 在生产客户端环境中正常工作
function sendConversion() {
  proxy.snaptr('track', 'PURCHASE', {
    currency: 'USD',
    price: 120.10,
    transaction_id: '11111'
  })
}
</script>

<template>
  <div>
    <button @click="sendConversion">
      发送转化事件
    </button>
  </div>
</template>
```

::