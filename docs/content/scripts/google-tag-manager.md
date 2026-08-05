---
title: Google Tag Manager
description: 加载 GTM Web 容器，并从 Nuxt 推送页面或同意事件。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-tag-manager.ts
    size: xs
---

[Google Tag Manager](https://marketingplatform.google.com/about/tag-manager/) 从 Web 容器加载标签，让你无需重新部署应用即可更改跟踪配置。

::callout
GTM 可以加载容器中配置的任何标签，因此其性能开销会因配置而异。如果你只需要 Google Analytics，[`useScriptGoogleAnalytics()`{lang="ts"}](/scripts/google-analytics){lang="ts"} 组合式函数可能更简单。
::

::callout{icon="i-heroicons-information-circle"}
Nuxt Scripts 只加载 GTM **容器**。跟踪功能来自 GTM 工作区中的标签和触发器，或来自你自己的 `dataLayer.push` 调用。若要自动跟踪页面、点击、滚动和视频，请为 GA4 Web 数据流启用 [GA4 增强型衡量](https://support.google.com/analytics/answer/9216061)。
::

::script-stats
::

::script-docs  
::  

### 发送页面事件

将代理与 [`useScriptEventPage()`{lang="ts"}](/docs/api/use-script-event-page){lang="ts"} 配合使用，在 Nuxt 完成初始客户端渲染或后续路由更改并更新页面标题后推送事件：

```ts
const { proxy } = useScriptGoogleTagManager({
  id: 'YOUR_ID' // 仅当你未全局配置时需要提供 id
})

useScriptEventPage(({ title, path }) => {
  // 在较早注册时对初始渲染运行，之后对路由更改运行。
  proxy.dataLayer.push({
    event: 'pageview',
    title,
    path
  })
})
```

## 同意模式

Google Tag Manager 接受 [GCMv2 同意状态](https://developers.google.com/tag-platform/security/guides/consent?consentmode=basic)。`defaultConsent` 会在 `gtm.js` 事件之前进入队列；对于后续选择，请使用 `consent.update()`{lang="ts"}。如果初始默认值由客户端状态决定，请在调用组合式函数之前解析该状态，并通过 `defaultConsent` 传入。初始化后调用 `consent.default()`{lang="ts"} 会在队列中加入另一个默认值，因此无法复现原始的顺序。

::callout{icon="i-heroicons-play" to="https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent" target="_blank"}
在 [StackBlitz](https://stackblitz.com) 上打开[Cookie 同意](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent)、[细粒度同意](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/granular-consent)或[区域同意](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/regional-consent)示例。
::

### 同意模式 v2 信号

| 信号             | 目的                     |
|------------------|--------------------------|
| `ad_storage`     | 广告相关的 Cookie          |
| `ad_user_data`   | 向 Google 发送用户数据用于广告 |
| `ad_personalization` | 个性化广告（再营销）        |
| `analytics_storage`| 分析相关的 Cookie          |

### 示例

```vue
<script setup lang="ts">
const { proxy, consent } = useScriptGoogleTagManager({
  id: 'GTM-XXXXXX',
  defaultConsent: {
    ad_storage: 'denied',
    ad_user_data: 'denied',
    ad_personalization: 'denied',
    analytics_storage: 'denied',
  },
})

function acceptAll() {
  consent.update({
    ad_storage: 'granted',
    ad_user_data: 'granted',
    ad_personalization: 'granted',
    analytics_storage: 'granted',
  })
}

function savePreferences(choices: { analytics: boolean, marketing: boolean }) {
  consent.update({
    analytics_storage: choices.analytics ? 'granted' : 'denied',
    ad_storage: choices.marketing ? 'granted' : 'denied',
    ad_user_data: choices.marketing ? 'granted' : 'denied',
    ad_personalization: choices.marketing ? 'granted' : 'denied',
  })
}

useScriptEventPage(({ title, path }) => {
  proxy.dataLayer.push({ event: 'pageview', title, path })
})
</script>
```

### 按地区的默认值

将一个数组传递给 `defaultConsent`，即可按照输入顺序为每个条目加入一条默认同意命令。这与 Google 的[特定区域同意模式](https://developers.google.com/tag-platform/security/guides/consent?consentmode=advanced#region-specific-behavior)一致：更具体的区域（例如 `US-CA`）会覆盖更宽泛的区域（`US`）；不包含 `region` 的条目是无作用域的全局回退值。

```vue
<script setup lang="ts">
useScriptGoogleTagManager({
  id: 'GTM-XXXXXX',
  defaultConsent: [
    {
      // 欧洲经济区 + 英国 + 瑞士：初始设为拒绝，并等待 500 毫秒获取选择。
      ad_storage: 'denied',
      ad_user_data: 'denied',
      ad_personalization: 'denied',
      analytics_storage: 'denied',
      region: ['AT', 'BE', 'BG', 'HR', 'CY', 'CZ', 'DK', 'EE', 'FI', 'FR', 'DE', 'GR', 'HU', 'IE', 'IT', 'LV', 'LT', 'LU', 'MT', 'NL', 'PL', 'PT', 'RO', 'SK', 'SI', 'ES', 'SE', 'GB', 'IS', 'LI', 'NO', 'CH'],
      wait_for_update: 500,
    },
    {
      // 其他所有地区：默认授予。
      ad_storage: 'granted',
      ad_user_data: 'granted',
      ad_personalization: 'granted',
      analytics_storage: 'granted',
    },
  ],
})
</script>
```

该模块会按输入顺序逐项原样转发。区域限定与非限定默认值之间的优先级由运行时的 gtag 决定，而不是由顺序决定。

`consent.update()`{lang="ts"} 和 `consent.default()`{lang="ts"} 都接受任意 `Partial<ConsentState>`{lang="ts"}；缺失的类别会保持其当前值。两种方法都会根据规范的 GCMv2 schema 验证输入，并在遇到未知键或非 `granted`/`denied` 值时通过 `consola` 发出警告。`onBeforeGtmStart` 仍然可用，作为任何其他 `gtm.start` 之前设置的通用逃生舱口（仅当 GTM ID 直接传递给组合式函数时可用，而不是通过 `nuxt.config` 传入时）。

::script-types
::

## 示例

### 服务端 GTM

使用[服务端标记](https://developers.google.com/tag-platform/tag-manager/server-side/intro)时，Web 容器仍在浏览器中运行，并将测量请求发送到由您运营的服务器容器。将每个受支持标记的传输网址（例如 Google 标记中的 `server_container_url`）设置为服务器容器；仅更改 GTM 加载器网址不会重新路由这些请求。

前提条件包括一个[服务端 GTM 容器](https://tagmanager.google.com)、[Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) 等托管服务或[手动部署](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide)，以及一个[自定义域名](https://developers.google.com/tag-platform/tag-manager/server-side/custom-domain)。

#### 配置

如果您配置的第一方标记域名提供 Web 容器加载器，请覆盖源地址并保留容器 ID 查询参数：

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleTagManager: {
        id: 'GTM-XXXXXX',
        trigger: 'onNuxtReady',
        scriptInput: {
          src: 'https://analytics.example.com/gtm.js?id=GTM-XXXXXX'
        }
      }
    }
  }
})
```

此源地址覆盖仅会更改浏览器加载 `gtm.js` 的位置。请按照 Google 的[服务端标记文档](https://developers.google.com/tag-platform/tag-manager/server-side)配置 Web 容器和服务器容器，将测量请求路由到服务器容器。
