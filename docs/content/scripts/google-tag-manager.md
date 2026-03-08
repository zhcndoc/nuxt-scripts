---

title: Google 标签管理器  
description: 在你的 Nuxt 应用中使用 Google 标签管理器。  
links:  
  - label: 源码  
    icon: i-simple-icons-github  
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-tag-manager.ts  
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
  id: 'YOUR_ID' // 仅当你未全局配置时需要提供id
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

::script-types  
::  

## 示例

### 服务器端 GTM 设置

服务器端 GTM 将标签执行搬到你的服务器，提高隐私性、性能（约快 500ms）及绕过广告拦截器。

**前提条件：** [服务器端 GTM 容器](https://tagmanager.google.com)、托管环境（[Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) / [Docker](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide)）以及自定义域名。

#### 配置

用你的自定义域名覆盖脚本源：

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

环境变量（`auth`、`preview`）可在 GTM 中找到：管理 > 环境 > 获取代码片段。

#### 故障排查

| 问题                         | 原因                         | 解决方案                                   |
|------------------------------|------------------------------|--------------------------------------------|
| 脚本被广告拦截器阻止           | 自定义域名被识别为跟踪器       | 使用不明显的子域名（避免使用 `gtm`、`analytics`、`tracking`） |
| Safari 中 Cookie 7 天后过期    | ITP 视子域为第三方            | 使用同源设置或实现 Cookie 持续器             |
| 预览模式无法使用               | 缺失或错误的 auth/preview 令牌 | 从 GTM：管理 > 环境 > 获取代码片段 获取正确令牌 |
| CORS 错误                    | 服务器容器配置错误             | 确保服务器容器允许你的域名请求               |
| `gtm.js` 返回 404             | 路径映射错误                   | 验证 CDN / 代理是否将 `/gtm.js` 路由到容器    |

有关基础设施设置，请参见 [Cloud Run](https://developers.google.com/tag-platform/tag-manager/server-side/cloud-run-setup-guide) 或 [Docker](https://developers.google.com/tag-platform/tag-manager/server-side/manual-setup-guide) 指南。

## 在 GTM 启动前配置

[`useScriptGoogleTagManager()`{lang="ts"}](/scripts/google-tag-manager){lang="ts"} 会自动初始化 Google 标签管理器，也会自动推送 `js`、`config` 和 `gtm.start` 事件。

如果你需要在 GTM 启动前配置，比如[设置同意模式](https://developers.google.com/tag-platform/security/guides/consent?consentmode=basic)，你有两个选项：

### 选项 1：在 nuxt.config 中使用 `defaultConsent`（推荐）

如果你在 `nuxt.config` 里配置 GTM，则使用 `defaultConsent` 选项。见上方[默认同意模式](#loading-globally)示例。

### 选项 2：使用 `onBeforeGtmStart` 回调

如果你在组件中直接调用 [`useScriptGoogleTagManager()`{lang="ts"}](/scripts/google-tag-manager){lang="ts"} 并传入 ID（不是在 nuxt.config 中配置），则可以使用 `onBeforeGtmStart` 钩子，该钩子会在推送 `gtm.start` 事件之前运行。

::callout{icon="i-heroicons-exclamation-triangle" color="warning"}  
`onBeforeGtmStart` 仅在直接传入 GTM ID 给 [`useScriptGoogleTagManager()`{lang="ts"}](/scripts/google-tag-manager){lang="ts"} 时生效，使用 nuxt.config 全局配置时该钩子无效，替代方案请用 `defaultConsent` 选项。  
::  

::callout{icon="i-heroicons-play" to="https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent" target="_blank"}  
在 [StackBlitz](https://stackblitz.com) 上试试实时的[Cookie 同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/cookie-consent)或[细粒度同意示例](https://stackblitz.com/github/nuxt/scripts/tree/main/examples/granular-consent)。  
::  

#### 同意模式 v2 信号

| 信号             | 目的                     |
|------------------|--------------------------|
| `ad_storage`     | 广告相关的 Cookie          |
| `ad_user_data`   | 向 Google 发送用户数据用于广告 |
| `ad_personalization` | 个性化广告（再营销）        |
| `analytics_storage`| 分析相关的 Cookie          |

#### 更新同意状态

当用户接受时，调用 `gtag('consent', 'update', ...)`{lang="ts"}：

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

若要在获得同意前阻止 GTM，可以结合使用 [`useScriptTriggerConsent()`{lang="ts"}](/docs/guides/consent){lang="ts"}。

```vue
<script setup lang="ts">
const consent = useState('consent', () => 'denied')

const { proxy } = useScriptGoogleTagManager({
  onBeforeGtmStart: (gtag) => {
    // 设置默认同意状态为 denied
    gtag('consent', 'default', {
      ad_user_data: 'denied',
      ad_personalization: 'denied',
      ad_storage: 'denied',
      analytics_storage: 'denied',
      wait_for_update: 500,
    })

    // 如果已经获得同意，则更新 gtag
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

// 将页面访问事件推送到 dataLayer
useScriptEventPage(({ title, path }) => {
  proxy.dataLayer.push({
    event: 'pageview',
    title,
    path
  })
})
</script>
```
