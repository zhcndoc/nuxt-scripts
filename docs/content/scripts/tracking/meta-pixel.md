---
title: Meta Pixel
description: 在你的 Nuxt 应用中使用 Meta Pixel。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/meta-pixel.ts
  size: xs
---

[Meta Pixel](https://www.facebook.com/business/tools/meta-pixel) 让你能够衡量、优化并为你的 Facebook 广告活动构建受众。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptMetaPixel`，方便你在 Nuxt 应用中集成 Meta Pixel。

### Nuxt 配置设置

在你的 Nuxt 应用中全局加载 Meta Pixel 最简单的方式是通过 Nuxt 配置。或者你也可以直接使用 [useScriptMetaPixel](#useScriptMetaPixel) 组合式函数。

如果你不打算发送自定义事件，可以利用[环境重写](https://nuxt.com/docs/getting-started/configuration#environment-overrides)来在开发环境禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      metaPixel: {
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
        metaPixel: {
          id: 'YOUR_ID',
        }
      }
    }
  }
})
```

::

#### 使用环境变量

如果你更喜欢通过环境变量配置你的 id。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      metaPixel: true,
    }
  },
  // 你需要提供一个运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        metaPixel: {
          id: '', // NUXT_PUBLIC_SCRIPTS_META_PIXEL_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_META_PIXEL_ID=<YOUR_ID>
```

## useScriptMetaPixel

`useScriptMetaPixel` 组合式函数让你精细控制 Meta Pixel 在你网站上的加载时机和方式。

```ts
const { proxy } = useScriptMetaPixel({
  id: 'YOUR_ID'
})
// 示例
proxy.fbq('track', 'ViewContent', {
  content_name: 'Nuxt Pixel',
  content_category: 'Nuxt',
})
```

请参考[注册脚本](/docs/guides/registry-scripts)指南了解更多高级用法。

### MetaPixelApi

```ts
export interface MetaPixelApi {
  fbq: FbqFns & {
    push: FbqFns
    loaded: boolean
    version: string
    queue: any[]
  }
  _fbq: MetaPixelApi['fbq']
}
type FbqArgs =
  | ['track', StandardEvents, EventObjectProperties?]
  | ['trackCustom', string, EventObjectProperties?]
  | ['trackSingle', string, StandardEvents, EventObjectProperties?]
  | ['trackSingleCustom', string, string, EventObjectProperties?]
  | ['init', string]
  | ['init', number, Record<string, any>?]
  | ['consent', ConsentAction]
  | [string, ...any[]]
type FbqFns = (...args: FbqArgs) => void
type StandardEvents = 'AddPaymentInfo' | 'AddToCart' | 'AddToWishlist' | 'CompleteRegistration' | 'Contact' | 'CustomizeProduct' | 'Donate' | 'FindLocation' | 'InitiateCheckout' | 'Lead' | 'Purchase' | 'Schedule' | 'Search' | 'StartTrial' | 'SubmitApplication' | 'Subscribe' | 'ViewContent'
interface EventObjectProperties {
  content_category?: string
  content_ids?: string[]
  content_name?: string
  content_type?: string
  contents?: { id: string, quantity: number }[]
  currency?: string
  delivery_category?: 'in_store' | 'curbside' | 'home_delivery'
  num_items?: number
  predicted_ltv?: number
  search_string?: string
  status?: 'completed' | 'updated' | 'viewed' | 'added_to_cart' | 'removed_from_cart' | string
  value?: number
  [key: string]: any
}
type ConsentAction = 'grant' | 'revoke'
```

### 配置结构

你必须在首次设置脚本时提供选项。

```ts
export const MetaPixelOptions = object({
  id: number(),
  sv: optional(number()),
})
```

## First-Party 模式

此脚本支持[第一方模式](/docs/guides/first-party)，通过你的域名路由所有流量，以提升隐私保护和绕过广告拦截。

通过在全局配置中启用 `scripts.firstParty: true`，该脚本将会：

- 从你的域名加载，而非 `connect.facebook.net`
- 通过你的服务器路由追踪请求（`/tr`）
- 全方位隐私匿名化 — IP、用户代理、语言、屏幕、时区和硬件指纹数据均被匿名化（不信任的广告网络）

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    firstParty: true,
    registry: {
      metaPixel: { id: '123456789' }
    }
  }
})
```

如果你想为该脚本单独禁用第一方模式：

```ts
useScriptMetaPixel({
  id: '123456789',
  scriptOptions: {
    firstParty: false // 直接从 Meta 加载
  }
})
```

## 示例

在生产环境中使用 Meta Pixel，并通过 `fbq` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptMetaPixel()

// 在开发和服务端渲染阶段无操作
// 在生产客户端正常工作
function sendConversion() {
  proxy.fbq('trackCustom', 'conversion', {
    value: 1,
    currency: 'USD'
  })
}
</script>

<template>
  <div>
    <button @click="sendConversion">
      发送转化
    </button>
  </div>
</template>
```

::
