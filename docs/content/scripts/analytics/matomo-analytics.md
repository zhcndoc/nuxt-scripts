---
title: Matomo 分析
description: 在你的 Nuxt 应用中使用 Matomo 分析。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/matomo-analytics.ts
    size: xs
---

[Matomo 分析](https://matomo.org/) 是一个适合 Nuxt 应用的优秀分析解决方案。

它可以提供关于你的网站表现、用户如何与你的内容互动以及他们如何在你的网站中导航的详细洞察。

在你的 Nuxt 应用中，全局加载 Matomo 分析最简单的方法是使用 Nuxt 配置。或者你也可以直接使用 [useScriptMatomoAnalytics](#useScriptMatomoAnalytics) 组合式函数。

## 全局加载

下面的配置假设你正在使用默认 `siteId` 为 `1` 的 Matomo 云版本。页面浏览量默认会在导航时**自动跟踪**。

如果你自托管，你需要提供 `matomoUrl`。如果你还有其他需要跟踪的网站，可以通过 `siteId` 添加。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      matomoAnalytics: {
        cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
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
        matomoAnalytics: {
          cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
        }
      }
    }
  }
})
```

```ts [环境变量]
export default defineNuxtConfig({
  scripts: {
    registry: {
      matomoAnalytics: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        matomoAnalytics: {
          // .env
          // NUXT_PUBLIC_SCRIPTS_MATOMO_ANALYTICS_CLOUD_ID=<your-id>
          cloudId: '', // NUXT_PUBLIC_SCRIPTS_MATOMO_ANALYTICS_CLOUD_ID
        },
      },
    },
  },
})
```

::

## useScriptMatomoAnalytics

`useScriptMatomoAnalytics` 组合式函数让你可以精细控制 Matomo 分析在你的网站何时以及如何加载。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
})
```

默认情况下，使用 `siteId` 为 `1`，且通过 `watch` 选项**自动启用**页面跟踪。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
  siteId: 2,
  // watch: true, // 默认启用 - 自动页面跟踪！
})
```

如果你想更灵活地控制跟踪，例如设置自定义维度，可以用 `proxy` 对象发送事件。

```ts
const { proxy } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID', // 例如 nuxt.matomo.cloud
})

// 设置自定义维度
proxy._paq.push(['setCustomDimension', 1, 'value'])
// 发送页面事件
proxy._paq.push(['trackPageView'])
```

请参阅 [配置 Schema](#config-schema) 获取所有可用选项。

## 自定义页面跟踪

默认情况下，所有页面都会自动跟踪，若你想禁用自动跟踪，可设置 `watch: false`。

```ts
import { useScriptEventPage } from '#nuxt-scripts'

const { proxy } = useScriptMatomoAnalytics({
  cloudId: 'YOUR_CLOUD_ID',
  watch: false, // 禁用自动跟踪
})

// 结合额外逻辑的自定义页面跟踪
useScriptEventPage((payload) => {
  // 根据路由设置自定义维度
  if (payload.path.startsWith('/products')) {
    proxy._paq.push(['setCustomDimension', 1, '产品页面'])
  }

  // 标准的 Matomo 跟踪调用（与内置的 watch 行为相同）
  proxy._paq.push(['setDocumentTitle', payload.title])
  proxy._paq.push(['setCustomUrl', payload.path])
  proxy._paq.push(['trackPageView'])

  // 跟踪额外的自定义事件
  proxy._paq.push(['trackEvent', 'Navigation', 'PageView', payload.path])
})
```

### 使用自托管 Matomo

对于自托管 Matomo，设置 `matomoUrl` 来自定义跟踪，如果你自定义了地址，可能还需要设置 `trackerUrl`。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  // 例如 https://your-url.com/tracker.js 和 https://your-url.com//matomo.php 都存在
  matomoUrl: 'https://your-url.com',
})
```

### 使用 Matomo 白标版本

对于 Matomo 白标版本，需要通过设置 `trackerUrl` 和 `scriptInput.src` 来自定义跟踪。

```ts
const matomoAnalytics = useScriptMatomoAnalytics({
  trackerUrl: 'https://c.staging.cookie3.co/lake',
  scriptInput: {
    src: 'https://cdn.cookie3.co/scripts/analytics/latest/cookie3.analytics.min.js',
  },
})
```

请参考 [Registry Scripts](/docs/guides/registry-scripts) 指南了解更多高级用法。

### MatomoAnalyticsApi

```ts
interface MatomoAnalyticsApi {
  _paq: unknown[]
}
```

### 配置 Schema

首次设置脚本时必须提供如下选项。

```ts
// matomoUrl 和 siteId 是必需的
export const MatomoAnalyticsOptions = object({
  matomoUrl: optional(string()),
  siteId: optional(union([string(), number()])),
  cloudId: optional(string()),
  trackerUrl: optional(string()),
  trackPageView: optional(boolean()), // 已废弃 - 请改用 watch
  enableLinkTracking: optional(boolean()),
  disableCookies: optional(boolean()),
  watch: optional(boolean()), // 默认：true
})
```

## 示例

仅在生产环境中使用 Matomo 分析，并通过 `_paq` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptMatomoAnalytics()

// 开发环境和 SSR 不起作用
// 仅在生产客户端生效
function sendConversion() {
  proxy._paq.push(['trackGoal', 1])
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