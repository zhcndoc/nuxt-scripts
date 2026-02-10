---
title: X Pixel
description: 在你的 Nuxt 应用中使用 X Pixel。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/x-pixel.ts
  size: xs
---

[X Pixel](https://x.com/) 让你能够收集、清理并控制你的客户数据。X 帮助你了解客户并个性化他们的体验。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptXPixel`，以便在你的 Nuxt 应用中轻松集成 X Pixel。

### Nuxt 配置设置

在你的 Nuxt 应用中全局加载 Meta Pixel 的最简单方式是使用 Nuxt 配置。或者，你也可以直接使用 [useScriptXPixel](#useScriptXPixel) 组合式函数。

如果你不打算发送自定义事件，可以使用 [环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 在开发环境中禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      xPixel: {
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
        xPixel: {
          id: 'YOUR_ID',
        }
      }
    }
  }
})
```

::

#### 使用环境变量

如果你更倾向于使用环境变量来配置你的 id。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      xPixel: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        xPixel: {
          id: '', // NUXT_PUBLIC_SCRIPTS_X_PIXEL_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_X_PIXEL_ID=<YOUR_ID>
```

## useScriptXPixel

`useScriptXPixel` 组合式函数让你可以精细控制 X Pixel 在你的网站上的加载时机和方式。

```ts
const { proxy } = useScriptXPixel({
  id: 'YOUR_ID'
})
// 示例
proxy.twq('event', '<EventId>', {
  value: 1,
})
```

请参阅[注册脚本](/docs/guides/registry-scripts)指南以了解更多高级用法。

### XPixelApi

```ts
export interface XPixelApi {
  twq: TwqFns & {
    loaded: boolean
    version: string
    queue: any[]
  }
}
type TwqFns =
  ((event: 'event', eventId: string, data?: EventObjectProperties) => void)
  & ((event: 'config', id: string) => void)
  & ((event: string, ...params: any[]) => void)
interface ContentProperties {
  content_type?: string | null
  content_id?: string | number | null
  content_name?: string | null
  content_price?: string | number | null
  num_items?: string | number | null
  content_group_id?: string | number | null
}
interface EventObjectProperties {
  value?: string | number | null
  currency?: string | null
  conversion_id?: string | number | null
  email_address?: string | null
  phone_number?: string | null
  contents: ContentProperties[]
}
```

### 配置模式

首次设置该脚本时必须提供选项。

```ts
export const XPixelOptions = object({
  id: string(),
  version: optional(string()),
})
```

## 示例

仅在生产环境中使用 X Pixel，同时使用 `twq` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptXPixel()

// 在开发环境和 SSR 中无操作
// 仅在生产环境客户端生效
function sendConversion() {
  proxy.twq('event', 'Purchase', {
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