---
title: Google 地图
description: 按需加载交互式地图、静态地图和位置搜索。
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

[Google 地图](https://maps.google.com/) 为网站提供交互式地图和静态地图。

Nuxt Scripts 提供了一个 [`useScriptGoogleMaps()`{lang="ts"}](/scripts/google-maps/api/use-script-google-maps){lang="ts"} 可组合函数，以及一个无头 [`<ScriptGoogleMaps>`{lang="html"}](/scripts/google-maps/api/script-google-maps){lang="html"} 组件，用于使用 Google 地图。

::script-types{exclude-components}
::

## 类型

安装 `@types/google.maps` 以获得完整的 TypeScript 支持。

```bash
pnpm add -D @types/google.maps
```

## 设置

在你的 `nuxt.config` 中启用 Google Maps，并通过环境变量提供你的 API 密钥：

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      // 注册基础设施，而不在每个页面上加载 Maps。
      // <ScriptGoogleMaps> 将在其元素触发器触发后加载它。
      googleMaps: {},
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_GOOGLE_MAPS_API_KEY=<你的_API_密钥>
```

仅当你希望交互式 Maps API 全局加载时，才在注册表条目中添加 `trigger: 'onNuxtReady'`。由于共享脚本实例已经在加载，这会绕过组件默认的交互延迟。

::callout{color="amber"}
Maps JavaScript API 和 Static Maps API 会将此密钥发送到浏览器。请遵循 Google 的 [API 安全指南](https://developers.google.com/maps/api-security-best-practices)：应用 Websites 应用限制，仅允许此网站使用的 API，并配置配额限制。运行时配置可以让部署值保持可配置；但它不会使 `NUXT_PUBLIC_` 密钥变为机密。
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
