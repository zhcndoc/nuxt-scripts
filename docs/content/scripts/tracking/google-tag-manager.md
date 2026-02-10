---
title: Google Tag Manager
description: 在你的 Nuxt 应用中使用 Google Tag Manager。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-tag-manager.ts
    size: xs
---

[Google Tag Manager](https://marketingplatform.google.com/about/tag-manager/) 是一个标签管理系统，允许你快速且轻松地更新网站或移动应用中的标签和代码片段，例如用于流量分析和营销优化的代码。

::callout
使用 Nuxt Scripts 你可能不需要 Google Tag Manager。GTM 体积为 82kb，会降低你网站的加载速度。
Nuxt Scripts 提供许多功能，你可以轻松在 Nuxt 应用中实现它们。如果你使用 GTM 来做 Google Analytics，可以改用 `useScriptGoogleAnalytics` 组合函数。
::

## 全局加载

如果你想避免在开发环境中加载分析脚本，可以在 Nuxt 配置中使用[环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides)。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleTagManager: {
        id: '<YOUR_ID>'
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
        googleTagManager: {
          id: '<YOUR_ID>',
        }
      }
    }
  }
})
```

```ts [默认同意模式]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleTagManager: {
        id: '<YOUR_ID>',
        defaultConsent: {
          // 根据 GTM 文档，这里可以是任意字符串或数字
          // 这里默认将所有同意类型设置为 'denied'（拒绝）
          'ad_user_data': 'denied',
          'ad_personalization': 'denied',
          'ad_storage': 'denied',
          'analytics_storage': 'denied',
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
      googleTagManager: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        googleTagManager: {
          // .env 文件中
          // NUXT_PUBLIC_SCRIPTS_GOOGLE_TAG_MANAGER_ID=<your-id>
          id: '',
        },
      },
    },
  },
})
```

::

## useScriptGoogleTagManager

`useScriptGoogleTagManager` 组合函数让你可以精细控制 Google Tag Manager 在网站上的加载时机和方式。

```ts
const { proxy } = useScriptGoogleTagManager({
  id: 'YOUR_ID' // 仅当你未在全局配置中设置时需要传入 id
})
// 例如
proxy.dataLayer.push({ event: 'conversion', value: 1 })
```

请参考[注册脚本指南](/docs/guides/registry-scripts)了解更多关于 `proxy` 的用法。

### 指南：发送页面事件

如果你想手动向 Google Tag Manager 发送页面事件，可以结合使用 `proxy` 和 [useScriptEventPage](/docs/api/use-script-event-page) 组合函数。
该组合会在路由变更且页面标题更新后触发你提供的函数。

```ts
const { proxy } = useScriptGoogleTagManager({
  id: 'YOUR_ID' // 仅当你未在全局配置中设置时需要传入 id
})

useScriptEventPage(({ title, path }) => {
  // 路由变更且标题更新后触发
  proxy.dataLayer.push({
    event: 'pageview',
    title,
    path
  })
})
```

### GoogleTagManagerApi

```ts
interface GoogleTagManagerApi {
  dataLayer: Record<string, any>[]
  google_tag_manager: GoogleTagManager
}
```

### 配置模式 (Config Schema)

首次设置该脚本时必须提供配置选项。

```ts
/**
 * GTM 配置选项，附带详细文档说明
 */
export const GoogleTagManagerOptions = object({
    /** GTM 容器 ID（格式：GTM-XXXXXX） */
    id: string(),

    /** 可选的数据层变量名 */
    l: optional(string()),

    /** 针对特定环境容器版本的认证令牌 */
    auth: optional(string()),

    /** 预览环境名称 */
    preview: optional(string()),

    /** 值为 true 时强制 GTM cookie 优先 */
    cookiesWin: optional(union([boolean(), literal('x')])),

    /** 值为 true 时开启调试模式 */
    debug: optional(union([boolean(), literal('x')])),

    /** 是否禁用个性化广告功能，值为 true 时禁用 */
    npa: optional(union([boolean(), literal('1')])),

    /** 自定义 dataLayer 名称（替代属性 "l"） */
    dataLayer: optional(string()),

    /** 环境专用容器的环境名称 */
    envName: optional(string()),

    /** 分析请求的引用策略 */
    authReferrerPolicy: optional(string()),
    
    /** GTM 默认同意设置 */
    defaultConsent: optional(record(string(), union([string(), number()]))),
  })
```

### 选项类型

```ts
type GoogleTagManagerInput = typeof GoogleTagManagerOptions & { onBeforeGtmStart?: (gtag: Gtag) => void }
```

## 示例

### 服务器端 GTM 设置

服务器端 GTM 将标签执行转移到服务器端，提升隐私、性能（约快 500ms）并绕过广告拦截器。

**前提条件：** [服务器端 GTM 容器](https://tagmanager.google.com)、主机服务（[Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) / [Docker](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide)）和自定义域名。

#### 配置

用你的自定义域覆盖脚本源地址：

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleTagManager: {
        id: 'GTM-XXXXXX',
        scriptInput: {
          src: 'https://gtm.example.com/gtm.js'
        }
      }
    }
  }
})
```

环境令牌（`auth`、`preview`）可在 GTM 路径：管理员 > 环境 > 获取代码段找到。

#### 故障排查

| 问题                       | 原因                          | 解决方案                                    |
|----------------------------|------------------------------|---------------------------------------------|
| 脚本被广告拦截器阻止         | 自定义域名被识别为跟踪器      | 使用不明显的子域名（避免使用 `gtm`、`analytics`、`tracking`） |
| Safari 中 cookie 7 天后过期 | ITP 将子域名视作第三方        | 使用同源设置或实现 cookie 保持机制          |
| 预览模式不可用             | 缺少或错误的 auth/preview 令牌 | 从 GTM 管理员 > 环境中复制令牌               |
| 发生 CORS 错误             | 服务器容器配置不正确          | 确保服务器容器允许来自你域名的请求          |
| `gtm.js` 返回 404          | 路径映射错误                  | 检查 CDN/代理是否将 `/gtm.js` 正确路由至容器 |

基础设施设置请参考 [Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) 或 [Docker](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide) 指南。

### 基础用法

仅在生产环境使用 Google Tag Manager，并通过 `dataLayer` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptGoogleTagManager()

// 在开发或 SSR 中不执行任何操作
// 仅在生产客户端正常工作
proxy.dataLayer.push({ event: 'conversion-step', value: 1 })
function sendConversion() {
  proxy.dataLayer.push({ event: 'conversion', value: 1 })
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

## GTM 启动前的配置

`useScriptGoogleTagManager` 会自动初始化 Google Tag Manager，即自动触发 `js`、`config` 和 `gtm.start` 事件。

如果你需要在 GTM 启动前做配置，比如[设置同意模式](https://developers.google.com/tag-platform/security/guides/consent?consentmode=basic)，你有两个选项：

### 选项 1：在 nuxt.config 中使用 `defaultConsent`（推荐）

如果你在 `nuxt.config` 中配置 GTM，使用 `defaultConsent` 选项。请参考上文的[默认同意模式](#loading-globally)示例。

### 选项 2：使用 `onBeforeGtmStart` 回调函数

如果你直接在组件中调用 `useScriptGoogleTagManager` （带 ID，非全局配置），使用 `onBeforeGtmStart` 钩子，该钩子会在推送 `gtm.start` 事件之前执行。

::callout{icon="i-heroicons-exclamation-triangle" color="warning"}
`onBeforeGtmStart` 仅在你直接传入 GTM ID 调用 `useScriptGoogleTagManager` 时生效，全局配置（nuxt.config）时无效。全局配置请使用 `defaultConsent` 选项。
::

::callout{icon="i-heroicons-play" to="https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent" target="_blank"}
试试 StackBlitz 上的[Cookie 同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent)或[细粒度同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/granular-consent)。
::

#### 同意模式 v2 信号

| 信号              | 用途                  |
|-------------------|-----------------------|
| `ad_storage`      | 广告用 Cookie          |
| `ad_user_data`    | 发送用户数据给 Google 广告 |
| `ad_personalization` | 个性化广告（再营销）      |
| `analytics_storage` | 分析用 Cookie          |

#### 同意状态更新

用户同意时调用 `gtag('consent', 'update', ...)`：

```ts
function acceptCookies() {
  window.gtag?.('consent', 'update', {
    ad_storage: 'granted',
    ad_user_data: 'granted',
    ad_personalization: 'granted',
    analytics_storage: 'granted',
  })
}
```

如需在用户同意之前完全阻止 GTM 加载，可配合 [useScriptTriggerConsent](/docs/guides/consent) 使用。

```vue
<script setup lang="ts">
const consent = useState('consent', () => 'denied')

const { proxy } = useScriptGoogleTagManager({
  onBeforeGtmStart: (gtag) => {
    // 设置默认同意状态为拒绝
    gtag('consent', 'default', {
      'ad_user_data': 'denied',
      'ad_personalization': 'denied',
      'ad_storage': 'denied',
      'analytics_storage': 'denied',
      'wait_for_update': 500,
    })

    // 如果用户已同意，更新 gtag 状态
    if (consent.value === 'granted') {
      gtag('consent', 'update', {
        ad_user_data: consent.value,
        ad_personalization: consent.value,
        ad_storage: consent.value,
        analytics_storage: consent.value
      })
    }
  }
})

// 推送页面浏览事件到 dataLayer
useScriptEventPage(({ title, path }) => {
  proxy.dataLayer.push({
    event: 'pageview',
    title,
    path
  })
})
</script>
```