---
title: Cloudflare Web Analytics
description: 在你的 Nuxt 应用中使用 Cloudflare Web Analytics。
links:
  - label: 源代码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/cloudflare-web-analytics.ts
    size: xs
---

[Cloudflare Web Analytics](https://developers.cloudflare.com/analytics/web-analytics/) 与 Nuxt 结合是一个极佳的隐私分析解决方案。它为您的网站提供免费的、以隐私为中心的分析。它不会收集访问者的个人数据，但仍能提供访问者体验中的网页性能详细洞察。

在 Nuxt 应用中全局加载 Cloudflare Web Analytics 最简单的方法是使用 Nuxt 配置。或者你也可以直接使用 [useScriptCloudflareWebAnalytics](#usescriptcloudflarewebanalytics) 组合函数。

## 全局加载

如果你想避免在开发环境中加载分析，可以在 Nuxt 配置中使用 [环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides)。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      cloudflareWebAnalytics: {
        token: 'YOUR_TOKEN_ID'
      }
    }
  }
})
```

```ts [仅限生产环境]
export default defineNuxtConfig({
  $production: {
    scripts: {
      registry: {
        cloudflareWebAnalytics: {
          token: 'YOUR_TOKEN_ID',
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
      cloudflareWebAnalytics: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        cloudflareWebAnalytics: {
          // .env 文件
          // NUXT_PUBLIC_SCRIPTS_CLOUDFLARE_WEB_ANALYTICS_TOKEN=<your-token>
          token: '', 
        },
      },
    },
  },
})
```

::

## useScriptCloudflareWebAnalytics

`useScriptCloudflareWebAnalytics` 组合函数让你对 Cloudflare Web Analytics 在网站上的加载时机和方式有更细粒度的控制。

```ts
function useScriptCloudflareWebAnalytics<T extends CloudflareWebAnalyticsApi>(_options?: CloudflareWebAnalyticsInput) {}
```

请参见 [Registry Scripts](/docs/guides/registry-scripts) 指南，了解更多高级用法。

该组合函数的默认配置包括：
- **触发时机：客户端** 脚本将在 Nuxt 完成水合时加载，以保持网页核心指标的准确性。

### CloudflareWebAnalyticsInput

```ts
export const CloudflareWebAnalyticsOptions = object({
  /**
   * Cloudflare Web Analytics 令牌。
   */
  token: string([minLength(32)]),
  /**
   * Cloudflare Web Analytics 通过覆盖 History API 的 pushState 方法并监听 onpopstate 自动测量 SPA。
   * 不支持基于哈希的路由。
   *
   * @default true
   */
  spa: optional(boolean()),
})
```

### CloudflareWebAnalyticsApi

```ts
export interface CloudflareWebAnalyticsApi {
  __cfBeacon: {
    load: 'single'
    spa: boolean
    token: string
  }
}
```

## 示例

通过 `app.vue` 在 Nuxt 准备好后加载 Cloudflare Web Analytics。

```vue [app.vue]
<script setup lang="ts">
useScriptCloudflareWebAnalytics({
  token: '12ee46bf598b45c2868bbc07a3073f58',
  scriptOptions: {
    trigger: 'onNuxtReady'
  }
})
</script>
```

Cloudflare Web Analytics 组合函数会将 `window.__cfBeacon` 对象注入全局作用域。如果你需要访问它，可以通过等待脚本加载来实现。

```ts
const { onLoaded } = useScriptCloudflareWebAnalytics()
onLoaded(({ cfBeacon }) => {
  console.log(cfBeacon)
})
```