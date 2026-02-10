---
title: Rybbit Analytics
description: 在您的 Nuxt 应用中使用 Rybbit Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/rybbit.ts
    size: xs
---

[Rybbit Analytics](https://www.rybbit.io/) 是一款注重隐私保护的分析解决方案，能够在不侵犯用户隐私的前提下跟踪您网站上的用户活动。

在您的 Nuxt 应用中全局加载 Rybbit Analytics 最简单的方法是使用 Nuxt 配置。或者，您也可以直接使用 [useScriptRybbitAnalytics](#useScriptRybbitAnalytics) 组合函数。

## 全局加载

如果您不打算发送自定义事件，可以使用 [环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 在开发环境中禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      rybbitAnalytics: {
        siteId: 'YOUR_SITE_ID'
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
        rybbitAnalytics: {
          siteId: 'YOUR_SITE_ID',
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
      rybbitAnalytics: true,
    }
  },
  // 您需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        rybbitAnalytics: {
          // .env 文件中
          // NUXT_PUBLIC_SCRIPTS_RYBBIT_ANALYTICS_SITE_ID=<your-site-id>
          siteId: ''
        },
      },
    },
  },
})
```

::

## useScriptRybbitAnalytics

`useScriptRybbitAnalytics` 组合函数允许您精细控制何时以及如何在您的网站上加载 Rybbit Analytics。

```ts
const rybbit = useScriptRybbitAnalytics({
  siteId: 'YOUR_SITE_ID'
})
```

请参考 [注册脚本](/docs/guides/registry-scripts) 指南，了解更多高级用法。

### 自托管 Rybbit Analytics

如果您使用的是自托管版本的 Rybbit Analytics，您可以提供自定义脚本源：

```ts
useScriptRybbitAnalytics({
  scriptInput: {
    src: 'https://your-rybbit-instance.com/api/script.js'
  },
  siteId: 'YOUR_SITE_ID'
})
```

### RybbitAnalyticsApi

```ts
export interface RybbitAnalyticsApi {
  /**
   * 跟踪页面浏览
   */
  pageview: () => void

  /**
   * 跟踪自定义事件
   * @param name 事件名称
   * @param properties 事件的可选属性
   */
  event: (name: string, properties?: Record<string, any>) => void

  /**
   * 为登录用户设置自定义用户 ID
   * @param userId 要设置的用户 ID（会存储在 localStorage 中）
   */
  identify: (userId: string) => void

  /**
   * 清除存储的用户 ID
   */
  clearUserId: () => void

  /**
   * 获取当前设置的用户 ID
   * @returns 当前用户 ID，未设置时返回 null
   */
  getUserId: () => string | null
  /**
   * @deprecated 请改用顶层函数
   */
  rybbit: RybbitAnalyticsApi
}
```

### 配置模式

首次设置脚本时，必须提供选项。

```ts
export const RybbitAnalyticsOptions = object({
  siteId: union([string(), number()]), // 必须项
  autoTrackPageview: optional(boolean()),
  trackSpa: optional(boolean()),
  trackQuery: optional(boolean()),
  trackOutbound: optional(boolean()),
  trackErrors: optional(boolean()),
  sessionReplay: optional(boolean()),
  webVitals: optional(boolean()),
  skipPatterns: optional(array(string())),
  maskPatterns: optional(array(string())),
  debounce: optional(number()),
  apiKey: optional(string()),
})
```

#### 配置选项说明

- `siteId`（必填）：您的 Rybbit Analytics 网站 ID
- `autoTrackPageview`：设置为 `false` 以禁用脚本加载时自动跟踪初始页面浏览。您需要手动调用 pageview 函数来跟踪页面浏览。默认值：`true`
- `trackSpa`：设置为 `false` 以禁用单页面应用的自动页面浏览跟踪
- `trackQuery`：设置为 `false` 以禁用 URL 查询字符串的跟踪
- `trackOutbound`：设置为 `false` 以禁用自动跟踪外链点击。默认值：`true`
- `trackErrors`：设置为 `true` 以启用 JavaScript 错误和未处理 Promise 拒绝的自动跟踪。仅跟踪同源错误，避免第三方脚本噪声。默认值：`false`
- `sessionReplay`：设置为 `true` 以启用会话重放录制。捕获用户交互、鼠标移动和 DOM 变更，用于调试和用户体验分析。默认值：`false`
- `webVitals`：设置为 `true` 以启用 Web Vitals 性能指标收集（LCP，CLS，INP，FCP，TTFB）。Web Vitals 默认禁用，以减少脚本大小和网络请求。默认值：`false`
- `skipPatterns`：忽略的 URL 路径模式数组
- `maskPatterns`：用于隐私保护的 URL 路径屏蔽模式数组
- `debounce`：在 URL 变化后延迟多少毫秒再跟踪页面浏览
- `apiKey`：开发时用于从 localhost 跟踪的 API 密钥。绕过自托管 Rybbit Analytics 实例的来源校验

## 示例

仅在生产环境使用 Rybbit Analytics 并跟踪自定义事件。

::code-group

```vue [EventTracking.vue]
<script setup lang="ts">
const { proxy } = useScriptRybbitAnalytics()

// 手动跟踪页面浏览
function trackPage() {
  proxy.pageview()
}

// 跟踪自定义事件
function trackEvent() {
  proxy.event('button_click', { buttonId: 'signup' })
}
</script>

<template>
  <div>
    <button @click="trackPage">
      跟踪自定义页面
    </button>
    <button @click="trackEvent">
      跟踪自定义事件
    </button>
  </div>
</template>
```

::