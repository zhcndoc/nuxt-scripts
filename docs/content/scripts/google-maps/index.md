---
title: 谷歌地图
description: 在你的 Nuxt 应用中展示性能优化的谷歌地图。
links:
  - label: useScriptGoogleMaps
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-maps.ts
    size: xs
  - label: "<ScriptGoogleMaps>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/GoogleMaps/ScriptGoogleMaps.vue
    size: xs
---

[谷歌地图](https://maps.google.com/) 允许你在网站中嵌入地图，并使用你的内容对其进行自定义。

Nuxt Scripts 提供了 [`useScriptGoogleMaps()`{lang="ts"}](/scripts/google-maps/api/use-script-google-maps){lang="ts"} 组合式函数和一个无头的 [`<ScriptGoogleMaps>`{lang="html"}](/scripts/google-maps/api/script-google-maps){lang="html"} 组件来与 Google Maps 交互。

::script-types{exclude-components}
::

## 类型

要在 Google Maps 中使用完整的 TypeScript 支持，你需要安装 `@types/google.maps` 依赖。

```bash
pnpm add -D @types/google.maps
```

## 设置

在你的 `nuxt.config` 中启用 Google Maps，并通过环境变量提供你的 API 密钥：

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleMaps: { trigger: 'onNuxtReady' },
    },
  },
  runtimeConfig: {
    public: {
      scripts: {
        googleMaps: {
          apiKey: '', // 对应环境变量 NUXT_PUBLIC_SCRIPTS_GOOGLE_MAPS_API_KEY
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_GOOGLE_MAPS_API_KEY=<你的_API_密钥>
```

你必须添加此配置。它会注册服务器代理路由，将你的 API 密钥保留在服务器端：
- `/_scripts/proxy/google-static-maps` 用于占位图片
- `/_scripts/proxy/google-maps-geocode` 用于位置搜索

::callout{color="amber"}
你可以直接在 `<ScriptGoogleMaps>`{lang="html"} 组件上传递 `api-key`，但不推荐这样做，因为这会在客户端请求中暴露你的密钥。
::

::callout{type="info"}
当你配置 `NUXT_SCRIPTS_PROXY_SECRET` 时，此脚本的代理端点会使用 [HMAC URL 签名](/docs/guides/first-party#proxy-endpoint-security)。有关设置说明，请参阅[安全指南](/docs/guides/first-party#proxy-endpoint-security)。
::

有关 API 费用和所需权限，请参阅 [计费与权限](/scripts/google-maps/guides/billing)。

## 快速开始

```vue
<template>
  <ScriptGoogleMaps
    :map-options="{
      center: { lat: -33.8688, lng: 151.2093 },
      zoom: 12,
    }"
  />
</template>
```

查看 [标记与信息窗口](/scripts/google-maps/guides/markers-and-info-windows) 指南了解如何添加标记、弹窗和自定义内容。查看 [形状与覆盖物](/scripts/google-maps/guides/shapes-and-overlays) 了解如何在地图上绘制图形。
