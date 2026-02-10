---
title: Databuddy Analytics
description: 在你的 Nuxt 应用中使用 Databuddy Analytics。
links:
  - label: 源代码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/databuddy-analytics.ts
    size: xs
---

[Databuddy](https://www.databuddy.cc/) 是一个隐私优先的分析平台，专注于性能和最小的数据收集。

使用 registry 可以轻松注入带有合理默认值的 Databuddy CDN 脚本，或者调用组合函数以实现细粒度控制。

## 全局加载

启用 Databuddy 全局的最简单方法是通过 `nuxt.config`（或模块配置）。你可以使用环境变量覆盖，仅在生产环境中启用。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      databuddyAnalytics: {
        clientId: 'YOUR_CLIENT_ID'
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
        databuddyAnalytics: {
          clientId: 'YOUR_CLIENT_ID'
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
      databuddyAnalytics: true,
    }
  },
  runtimeConfig: {
    public: {
      scripts: {
        databuddyAnalytics: {
          // .env
          // NUXT_PUBLIC_SCRIPTS_DATABUDDY_ANALYTICS_CLIENT_ID=<your-client-id>
          clientId: ''
        },
      },
    },
  },
})
```

::

## useScriptDatabuddyAnalytics

`useScriptDatabuddyAnalytics` 组合函数让你控制何时以及如何加载 Databuddy。

```ts
const db = useScriptDatabuddyAnalytics({
  clientId: 'YOUR_CLIENT_ID',
  trackWebVitals: true,
  trackErrors: true,
  enableBatching: true,
})
```

该组合函数返回脚本代理（如果可用）。你可以通过 `db` 或者 `window.db` / `window.databuddy` 与全局 API 交互。

### CDN / 自托管

默认情况下，registry 会注入 `https://cdn.databuddy.cc/databuddy.js`。如果你自行托管脚本，可以通过选项传入 `scriptUrl` 来覆盖 `src`。

```ts
useScriptDatabuddyAnalytics({
  scriptInput: { src: 'https://my-host/databuddy.js' },
  clientId: 'YOUR_CLIENT_ID'
})
```

### DatabuddyAnalyticsApi

```ts
export interface DatabuddyAnalyticsApi {
  track: (eventName: string, properties?: Record<string, any>) => Promise<any> | any | void
  screenView: (path?: string, properties?: Record<string, any>) => void
  setGlobalProperties: (properties: Record<string, any>) => void
  trackCustomEvent: (eventName: string, properties?: Record<string, any>) => void
  clear: () => void
  flush: () => void
}
```

### 配置模式

首次配置 registry 时，必须提供 `clientId`。registry 支持大量传递给脚本的 Databuddy 选项，这些选项通过 `data-` 属性传递。

```ts
export const DatabuddyAnalyticsOptions = object({
  clientId: string(),
  scriptUrl: optional(string()),
  apiUrl: optional(string()),
  disabled: optional(boolean()),
  trackScreenViews: optional(boolean()),
  trackPerformance: optional(boolean()),
  trackSessions: optional(boolean()),
  trackWebVitals: optional(boolean()),
  trackErrors: optional(boolean()),
  trackOutgoingLinks: optional(boolean()),
  trackScrollDepth: optional(boolean()),
  trackEngagement: optional(boolean()),
  trackInteractions: optional(boolean()),
  trackAttributes: optional(boolean()),
  trackHashChanges: optional(boolean()),
  trackExitIntent: optional(boolean()),
  trackBounceRate: optional(boolean()),
  enableBatching: optional(boolean()),
  batchSize: optional(number()),
  batchTimeout: optional(number()),
  enableRetries: optional(boolean()),
  maxRetries: optional(number()),
  initialRetryDelay: optional(number()),
  samplingRate: optional(number()),
  sdk: optional(string()),
  sdkVersion: optional(string()),
  enableObservability: optional(boolean()),
  observabilityService: optional(string()),
  observabilityEnvironment: optional(string()),
  observabilityVersion: optional(string()),
  enableLogging: optional(boolean()),
  enableTracing: optional(boolean()),
  enableErrorTracking: optional(boolean()),
})
```

## 示例

使用组合函数代理跟踪自定义事件（在 SSR / 开发环境中为无操作）：

::code-group

```vue [EventButton.vue]
<script setup lang="ts">
const { proxy } = useScriptDatabuddyAnalytics({ clientId: 'YOUR_CLIENT_ID' })

function sendEvent() {
  proxy?.track('signup_completed', { plan: 'pro' })
}
</script>

<template>
  <button @click="sendEvent">发送事件</button>
</template>
```
