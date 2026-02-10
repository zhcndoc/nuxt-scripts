---
title: Reddit 像素
description: 在你的 Nuxt 应用中使用 Reddit 像素。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/reddit-pixel.ts
  size: xs
---

[Reddit 像素](https://advertising.reddithelp.com/en/categories/custom-audiences-and-conversion-tracking/reddit-pixel) 帮助你跟踪转化并为你的 Reddit 广告活动构建受众。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptRedditPixel`，可让你轻松地在 Nuxt 应用中集成 Reddit 像素。

### Nuxt 配置设置

在你的 Nuxt 应用中全局加载 Reddit 像素的最简单方式是使用 Nuxt 配置。或者，你也可以直接使用 [useScriptRedditPixel](#useScriptRedditPixel) 组合式函数。

如果你不打算发送自定义事件，可以使用 [环境变量覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 来在开发环境中禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      redditPixel: {
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
        redditPixel: {
          id: 'YOUR_ID',
        }
      }
    }
  }
})
```

::

#### 使用环境变量

如果你更喜欢通过环境变量来配置你的 id。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      redditPixel: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        redditPixel: {
          id: '', // NUXT_PUBLIC_SCRIPTS_REDDIT_PIXEL_ID
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_REDDIT_PIXEL_ID=<YOUR_ID>
```

## useScriptRedditPixel

`useScriptRedditPixel` 组合式函数让你可以精细控制 Reddit 像素在你的站点上何时以及如何加载。

```ts
const { proxy } = useScriptRedditPixel({
  id: 'YOUR_ID'
})
// 示例
proxy.rdt('track', 'Lead')
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南以了解更多高级用法。

### RedditPixelApi

```ts
export interface RedditPixelApi {
  rdt: RdtFns & {
    sendEvent: (rdt: RedditPixelApi['rdt'], args: unknown[]) => void
    callQueue: unknown[]
  }
}
type RdtFns
  = & ((event: 'init', id: string) => void)
    & ((event: 'track', eventName: string) => void)
```

### 配置模式

首次设置脚本时，你必须提供选项。

```ts
export const RedditPixelOptions = object({
  id: string(),
})
```

## 示例

仅在生产环境使用 Reddit 像素，同时使用 `rdt` 发送跟踪事件。

::code-group

```vue [TrackingButton.vue]
<script setup lang="ts">
const { proxy } = useScriptRedditPixel()

// 在开发环境和 SSR 中无操作
// 仅在生产环境客户端有效
function trackConversion() {
  proxy.rdt('track', 'Lead')
}
</script>

<template>
  <div>
    <button @click="trackConversion">
      跟踪转化
    </button>
  </div>
</template>
```

::