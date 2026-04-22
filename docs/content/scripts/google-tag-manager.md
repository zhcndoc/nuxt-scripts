---
title: Google Tag Manager
description: 在你的 Nuxt 应用中使用 Google 标签管理器。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-tag-manager.ts
    size: xs
---

[Google 标签管理器](https://marketingplatform.google.com/about/tag-manager/) 是一个标签管理系统，允许你快速且轻松地更新网站或移动应用上的标签和代码片段，例如用于流量分析和营销优化的标签。

::callout  
你可能不需要在 Nuxt Scripts 中使用 Google 标签管理器。GTM 大小为 82kb，会降低你网站的速度。  
Nuxt Scripts 提供了许多功能，可以轻松地在 Nuxt 应用内实现。如果你使用 GTM 来做 Google Analytics，可以使用 [`useScriptGoogleAnalytics()`{lang="ts"}](/scripts/google-analytics){lang="ts"} 组合式函数替代。  
::  

::script-stats  
::  

::script-docs  
::  

### 指南：发送页面事件

如果你想手动将页面事件发送到 Google 标签管理器，可以使用 `proxy` 搭配 [`useScriptEventPage()`{lang="ts"}](/docs/api/use-script-event-page){lang="ts"} 组合式函数。  
该函数会在 Nuxt 更新页面标题后，页面路由变更时触发所提供的函数。

```ts
const { proxy } = useScriptGoogleTagManager({
  id: 'YOUR_ID' // 仅当你未全局配置时需要提供 id
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

## 同意模式

Google Tag Manager 原生支持 [GCMv2 同意状态](https://developers.google.com/tag-platform/security/guides/consent?consentmode=basic)。使用 `defaultConsent` 设置默认值（会在 `gtm.js` 事件之前将 `['consent','default', state]` 推送到 dataLayer 中），并在运行时调用 `consent.update()`{lang="ts"}。

::callout{icon="i-heroicons-play" to="https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent" target="_blank"}  
在 [StackBlitz](https://stackblitz.com) 上试试实时的 [Cookie 同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent) 或 [细粒度同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/granular-consent)。  
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

`onBeforeGtmStart` 仍然可用，作为任何其他 `gtm.start` 之前设置的通用逃生通道（仅当 GTM ID 直接传给组合式函数时可用，而不是通过 `nuxt.config` 传入）。

::script-types
::

## 示例

### 服务器端 GTM 设置

服务器端 GTM 将标签执行转移到你的服务器，以获得更好的隐私性、性能（快约 500 毫秒）并绕过广告拦截器。

**前提条件：** [服务器端 GTM 容器](https://tagmanager.google.com)、托管服务（[Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) / [Docker](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide)）以及自定义域名。

#### 配置

使用你的自定义域名覆盖脚本源：

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

对于环境令牌（`auth`、`preview`），请在 GTM 中查找：管理 > 环境 > 获取代码片段。

#### 故障排除

| 问题 | 原因 | 解决方案 |
|-------|-------|----------|
| 脚本被广告拦截器阻止 | 自定义域名被检测为跟踪器 | 使用不显眼的子域名（避免使用 `gtm`、`analytics`、`tracking`） |
| Safari 中 Cookie 在 7 天后过期 | ITP 将子域名视为第三方 | 使用同源设置或实施 Cookie 保持器 |
| 预览模式不工作 | 缺少或错误的 auth/preview 令牌 | 从 GTM 复制令牌：管理 > 环境 > 获取代码片段 |
| CORS 错误 | 服务器容器配置错误 | 确保你的服务器容器允许来自你域名的请求 |
| `gtm.js` 返回 404 | 路径映射错误 | 验证你的 CDN/代理是否将 `/gtm.js` 路由到容器 |

有关基础设施设置，请参阅 [Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) 或 [Docker](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide) 指南。
