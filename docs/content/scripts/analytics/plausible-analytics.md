---
title: Plausible Analytics
description: 在您的 Nuxt 应用中使用 Plausible Analytics。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/plausible-analytics.ts
    size: xs
---

[Plausible Analytics](https://plausible.io/) 是一个注重隐私保护的 Nuxt 应用分析解决方案，允许您在不侵犯用户隐私的情况下跟踪网站流量。

在 Nuxt 应用中全局加载 Plausible Analytics 的最简单方式是使用 Nuxt 配置。或者，您也可以直接使用 [useScriptPlausibleAnalytics](#useScriptPlausibleAnalytics) 组合函数。

## 全局加载

如果您不打算发送自定义事件，可以使用 [环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 在开发环境中禁用脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      plausibleAnalytics: {
        // 从您的 Plausible 脚本 URL 获取：
        // https://plausible.io/js/pa-gYyxvZhkMzdzXBAtSeSNz.js
        //                         ^^^^^^^^^^^^^^^^^^^^^^^^^^
        scriptId: 'gYyxvZhkMzdzXBAtSeSNz'
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
        plausibleAnalytics: {
          scriptId: 'YOUR_SCRIPT_ID',
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
      plausibleAnalytics: true,
    }
  },
  // 需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        plausibleAnalytics: {
          // .env
          // NUXT_PUBLIC_SCRIPTS_PLAUSIBLE_ANALYTICS_SCRIPT_ID=<your-script-id>
          scriptId: ''
        },
      },
    },
  },
})
```

::

## useScriptPlausibleAnalytics

`useScriptPlausibleAnalytics` 组合函数让您可以精细控制 Plausible Analytics 在网站上的加载时机和方式。

```ts
// 2025年10月更新格式
const plausible = useScriptPlausibleAnalytics({
  // 从：https://plausible.io/js/pa-gYyxvZhkMzdzXBAtSeSNz.js 提取
  //                                         ^^^^^^^^^^^^^^^^^^^^^^^^^^
  scriptId: 'gYyxvZhkMzdzXBAtSeSNz'
})
```

请参阅 [Registry Scripts](/docs/guides/registry-scripts) 指南以了解更多高级用法。

### 自托管 Plausible

如果您使用自托管版本的 Plausible，需要提供脚本的明确 src 以确保事件 API 发送到正确的端点。

```ts
useScriptPlausibleAnalytics({
  scriptInput: {
    src: 'https://my-self-hosted-plausible.io/js/script.js'
  }
})
```

### PlausibleAnalyticsApi

```ts
export interface PlausibleAnalyticsApi {
  plausible: ((event: '404', options: Record<string, any>) => void) &
  ((event: 'event', options: Record<string, any>) => void) &
  ((...params: any[]) => void) & {
    q: any[]
  }
}
```

### 配置方案

首次设置脚本时必须提供选项。

```ts
export interface PlausibleAnalyticsOptions {
  /**
   * 您网站的唯一脚本 ID（推荐 - 2025年10月新格式）
   * 从：<script src="https://plausible.io/js/pa-{scriptId}.js"></script> 提取
   */
  scriptId?: string
  /** 每个页面浏览都要跟踪的自定义属性 */
  customProperties?: Record<string, any>
  /** 自定义跟踪端点 URL */
  endpoint?: string
  /** 配置文件下载跟踪 */
  fileDownloads?: {
    fileExtensions?: string[]
  }
  /** 为单页应用启用基于哈希的路由 */
  hashBasedRouting?: boolean
  /** 设置为 false 时需手动触发页面浏览事件 */
  autoCapturePageviews?: boolean
  /** 启用本地主机跟踪 */
  captureOnLocalhost?: boolean
  /** 启用表单提交跟踪 */
  trackForms?: boolean
}
```

```ts
export interface PlausibleAnalyticsDeprecatedOptions {
  /**
   * 您的网站域名
   * @deprecated 请使用 `scriptId`（2025年10月新格式）代替
   */
  domain?: string
  /**
   * 用于额外功能的脚本扩展
   * @deprecated 请改用如 `hashBasedRouting`、`captureOnLocalhost` 等初始化选项
   */
  extension?: 'hash' | 'outbound-links' | 'file-downloads' | 'tagged-events' | 'revenue' | 'pageview-props' | 'compat' | 'local' | 'manual'
}
 
```

**注意：** `scriptId` 可以在您的 Plausible 仪表盘中，网站设置的 **网站安装** 部分找到。

**提取您的 Script ID：**

Plausible 提供给您的脚本标签如下：

```html
<script async src="https://plausible.io/js/pa-gYyxvZhkMzdzXBAtSeSNz.js"></script>
```

您的 `scriptId` 是 `pa-` 和 `.js` 之间的部分：

```ts
scriptId: 'gYyxvZhkMzdzXBAtSeSNz'
//         ^^^^^^^^^^^^^^^^^^^^^^^
//         从 pa-{scriptId}.js 中提取
```

## 示例

只在生产环境使用 Plausible Analytics，同时使用 `plausible` 发送转换事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptPlausibleAnalytics()

// 开发和服务端渲染时无操作
// 生产客户端时正常工作
proxy.plausible('event', { name: 'conversion-step' })
function sendConversion() {
  proxy.plausible('event', { name: 'conversion' })
}
</script>

<template>
  <div>
    <button @click="sendConversion">
      发送转换事件
    </button>
  </div>
</template>
```

::
