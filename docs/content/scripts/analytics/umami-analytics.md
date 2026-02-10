---
title: Umami Analytics
description: 在你的 Nuxt 应用中使用 Umami Analytics。
links:
  - label: 源代码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/umami-analytics.ts
    size: xs
---

[Umami](https://umami.is/) 收集你关心的所有指标，帮助你做出更好的决策。

在你的 Nuxt 应用中全局加载 Umami Analytics 的最简单方式是使用 Nuxt 配置。或者你也可以直接使用 [useScriptUmamiAnalytics](#useScriptUmamiAnalytics) 组合式函数。

## 全局加载

如果你不打算发送自定义事件，可以使用[环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides)来在开发环境禁用脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      umamiAnalytics: {
        websiteId: '你的_WEBSITE_ID'
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
        umamiAnalytics: {
          websiteId: '你的_WEBSITE_ID',
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
      umamiAnalytics: true,
    }
  },
  // 你需要提供运行时配置来访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        umamiAnalytics: {
          // .env 配置
          // NUXT_PUBLIC_SCRIPTS_UMAMI_ANALYTICS_WEBSITE_ID=<你的 websiteId>
          websiteId: ''
        },
      },
    },
  },
})
```

::

## useScriptUmamiAnalytics

`useScriptUmamiAnalytics` 组合式函数让你可以精细地控制 Umami Analytics 在你网站上的加载时机和方式。

```ts
const umami = useScriptUmamiAnalytics({
  websiteId: '你的_WEBSITE_ID'
})
```

请参阅 [注册脚本指南](/docs/guides/registry-scripts) 了解更多高级用法。

### 自托管 Umami

如果你使用的是自托管版本的 Umami，则需要显式提供脚本的 `src`，确保 API 事件发送到正确的端点。

```ts
useScriptUmamiAnalytics({
  scriptInput: {
    src: 'https://my-self-hosted/script.js'
  }
})
```

### UmamiAnalyticsApi

```ts
export interface UmamiAnalyticsApi {
  track: ((payload?: Record<string, any>) => void) &((event_name: string, event_data: Record<string, any>) => void)
  identify: (session_data?: Record<string, any> | string) => void
}
```

### 配置模式

首次设置脚本时必须提供以下选项。

```ts
export const UmamiAnalyticsOptions = object({
  websiteId: string(), // 必填
  /**
   * 默认情况下，Umami 会将数据发送到脚本所在的位置。
   * 你可以通过此项重写数据发送到其他地址。
   */
  hostUrl: optional(string()),
  /**
   * 默认情况下，Umami 会自动跟踪所有页面访问和事件。
   * 你可以禁用此功能，手动使用跟踪器函数跟踪事件。
   * https://umami.is/docs/tracker-functions
   */
  autoTrack: optional(boolean()),
  /**
   * 如果你希望跟踪器只在特定域名下运行，可以将它们添加到跟踪器脚本中。
   * 这是以逗号分隔的域名列表。
   * 对于处于预发布或开发环境时非常有用。
   */
  domains: optional(array(string())),
  /**
   * 如果你希望跟踪器在特定标签下收集事件。
   * 仪表盘可以基于标签过滤事件。
   */
  tag: optional(string()),
  /**
   * 发送数据到 Umami 前调用的函数。
   * 该函数接收两个参数：type 和 payload。
   * 返回 payload 继续发送，返回假值则取消发送。
   */
  beforeSend: optional(union([
    custom<(type: string, payload: Record<string, any>) => Record<string, any> | null | false>(input => typeof input === 'function'),
    string(),
  ])),
})
```

## 高级功能

### 会话识别

Umami v2.18.0 以上支持通过 `identify` 函数设置唯一的会话 ID。你可以传入字符串（唯一 ID）或包含会话数据的对象：

```ts
const { proxy } = useScriptUmamiAnalytics({
  websiteId: '你的_WEBSITE_ID'
})

// 使用唯一字符串 ID
proxy.identify('user-12345')

// 使用会话数据对象
proxy.identify({
  userId: 'user-12345',
  plan: 'premium'
})
```

### 使用 beforeSend 进行数据过滤

`beforeSend` 选项允许你检查、修改或取消发送到 Umami 的数据。这对于实现自定义隐私控制或数据过滤非常有用：

```ts
useScriptUmamiAnalytics({
  websiteId: '你的_WEBSITE_ID',
  beforeSend: (type, payload) => {
    // 打印发送数据（用于调试）
    console.log('发送到 Umami:', type, payload)
    
    // 过滤包含敏感数据的网址
    if (payload.url && payload.url.includes('private')) {
      return false // 取消发送
    }
    
    // 修改发送数据
    return {
      ...payload,
      referrer: '' // 隐藏来源页以保护隐私
    }
  }
})
```

你也可以提供全局定义函数的函数名字符串：

```ts
// 全局定义函数
window.myBeforeSendHandler = (type, payload) => {
  return checkPrivacyRules(payload) ? payload : false
}

useScriptUmamiAnalytics({
  websiteId: '你的_WEBSITE_ID',
  beforeSend: 'myBeforeSendHandler'
})
```

## 示例

仅在生产环境使用 Umami Analytics，并通过 `track` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptUmamiAnalytics()

// 开发环境和 SSR 下为 noop
// 仅在生产环境客户端正常工作
proxy.track('event', { name: 'conversion-step' })

function sendConversion() {
  proxy.track('event', { name: 'conversion' })
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