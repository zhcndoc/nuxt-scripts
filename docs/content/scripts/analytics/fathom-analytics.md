---
title: Fathom Analytics
description: 在你的 Nuxt 应用中使用 Fathom Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/fathom-analytics.ts
    size: xs
---

[Fathom Analytics](https://usefathom.com/) 是一款适用于你的 Nuxt 应用的出色隐私分析解决方案。它不会收集访客的个人数据，却能提供详细的站点使用情况洞察。

## 全局加载

在你的 Nuxt 应用中全局加载 Fathom Analytics 最简单的方法是使用 Nuxt 配置，提供你的站点 ID 字符串。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      fathomAnalytics: {
        site: 'YOUR_TOKEN_ID'
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
        fathomAnalytics: {
          site: 'YOUR_SITE_ID'
        }
      }
    }
  },
})
```


```ts [环境变量]
export default defineNuxtConfig({
  scripts: {
    registry: {
      fathomAnalytics: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        fathomAnalytics: {
          // .env
          // NUXT_PUBLIC_SCRIPTS_FATHOM_ANALYTICS_SITE=<your-token>
          token: '',
        },
      },
    },
  },
})
```

::

## 组合函数 `useScriptFathomAnalytics`

`useScriptFathomAnalytics` 组合函数让你可以细粒度地控制 Fathom Analytics 在你站点上的加载时机和方式。

```ts
useScriptFathomAnalytics(options)
```

## 默认值

- **触发时机**：脚本将在 Nuxt 被水合（hydrated）时加载。

## 选项

```ts
export const FathomAnalyticsOptions = object({
  /**
   * Fathom Analytics 的站点 ID。
   */
  site: string(),
  /**
   * Fathom Analytics 的跟踪模式。
   */
  spa: optional(union([literal('auto'), literal('history'), literal('hash')])),
  /**
   * 是否自动跟踪页面浏览。
   */
  auto: optional(boolean()),
  /**
   * 启用规范 URL 跟踪。
   */
  canonical: optional(boolean()),
  /**
   * 遵守“请勿追踪”请求。
   */
  honorDnt: optional(boolean()),
})
```

此外，和所有注册脚本一样，你可以提供额外配置：

- `scriptInput` - 要添加到 script 标签的 HTML 属性。
- `scriptOptions` - 详见 [Script Options]。不支持打包，`bundle: true` 不会生效。

## 返回值

Fathom Analytics 组合函数会向全局范围注入一个 `window.fathom` 对象。

```ts
export interface FathomAnalyticsApi {
  beacon: (ctx: { url: string, referrer?: string }) => void
  blockTrackingForMe: () => void
  enableTrackingForMe: () => void
  isTrackingEnabled: () => boolean
  send: (type: string, data: unknown) => void
  setSite: (siteId: string) => void
  sideId: string
  trackPageview: (ctx?: { url: string, referrer?: string }) => void
  trackGoal: (goalId: string, cents: number) => void
  trackEvent: (eventName: string, value: { _value: number }) => void
}
```

你可以直接通过代理访问 `fathom` 对象，或者等待 `$script` Promise 以访问该对象。建议对于无返回值的函数使用代理。

::code-group

```ts [代理]
const { proxy } = useScriptFathomAnalytics()
function trackMyGoal() {
  proxy.trackGoal('MY_GOAL_ID', 100)
}
```

```ts [加载完成时]
const { onLoaded } = useScriptFathomAnalytics()
onLoaded(({ trackGoal }) => {
  trackGoal('MY_GOAL_ID', 100)
})
```

::

## 示例

通过 `app.vue` 在 Nuxt 准备好时加载 Fathom Analytics。

```vue [app.vue]
<script setup>
useScriptFathomAnalytics({
  site: 'YOUR_SITE_ID',
  scriptOptions: {
    trigger: 'onNuxtReady'
  }
})
</script>
```