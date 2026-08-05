---
title: MapLibre GL JS
description: 使用 MapLibre GL JS 将延迟加载、与提供商无关的矢量地图添加到 Nuxt 中。
links:
  - label: useScriptMapLibre
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/maplibre.ts
    size: xs
  - label: "<ScriptMapLibreMap>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/MapLibre/ScriptMapLibreMap.vue
    size: xs
---

[MapLibre GL JS](https://maplibre.org/maplibre-gl-js/docs/) 是一个用于交互式矢量地图的开源 WebGL 渲染器。它负责渲染地图并处理交互；样式、瓦片源、归属信息、地理编码和路径规划服务则由你分别选择。

Nuxt Scripts 通过 [`useScriptMapLibre()`{lang="ts"}](/scripts/maplibre/api/use-script-maplibre) 组合式函数，以及用于常见地图资源的声明式组件，支持 MapLibre GL JS 5.24。第一方模式可以打包 JavaScript SDK。默认情况下，Nuxt Scripts 会单独加载所需的 MapLibre 样式表。

::script-types{exclude-components}
::

## 设置

安装 MapLibre 以获取其 TypeScript 类型定义，然后启用注册表条目：

```bash
pnpm add -D maplibre-gl
```

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      maplibre: { trigger: false },
    },
  },
})
```

## 快速开始

传入 MapLibre 样式 URL 或内联样式规范。本示例使用 OpenFreeMap 的免密钥 Liberty 样式。

```vue
<script setup lang="ts">
import type { LngLatLike } from 'maplibre-gl'

const center = ref<LngLatLike>([144.9631, -37.8136])
</script>

<template>
  <ScriptMapLibreMap
    v-model:center="center"
    map-style="https://tiles.openfreemap.org/styles/liberty"
    :zoom="12"
    width="100%"
    :height="480"
    aria-label="墨尔本市中心地图"
  >
    <template #description>
      墨尔本中央商务区的交互式街道地图。
    </template>

    <ScriptMapLibreNavigationControl position="top-right" />

    <ScriptMapLibreMarker
      :position="center"
      aria-label="墨尔本中央商务区"
    >
      <ScriptMapLibrePopup>
        墨尔本中央商务区
      </ScriptMapLibrePopup>
    </ScriptMapLibreMarker>
  </ScriptMapLibreMap>
</template>
```

MapLibre 坐标使用 `[经度, 纬度]` 顺序。这与 Leaflet 使用的 `[纬度, 经度]` 顺序相反。

默认的 `visible` 触发器会将 SDK、样式和地图切片请求延迟到地图接近视口时。该组件会在 SSR 期间预留尺寸，以避免布局偏移。

::callout{color="amber"}
OpenFreeMap 的公共实例无需 API 密钥，但不提供 SLA。在选择生产环境中的样式服务之前，请阅读[样式与提供商](/scripts/maplibre/guides/styles-and-providers)。
::

::callout{icon="i-lucide-route"}
**真实案例：**使用响应式路线进度、实时配送员标记、错误回退内容和无障碍状态替代方案，构建[配送跟踪器](/scripts/maplibre/guides/delivery-tracker)。
::

## 组件

- [`<ScriptMapLibreMap>`{lang="html"}](/scripts/maplibre/api/script-maplibre-map) 创建地图并控制延迟加载。
- [`<ScriptMapLibreMarker>`{lang="html"}](/scripts/maplibre/api/marker) 添加可访问且具有响应性的标记。
- [`<ScriptMapLibrePopup>`{lang="html"}](/scripts/maplibre/api/popup) 将插槽中的 HTML 绑定到标记或坐标。
- [`<ScriptMapLibreNavigationControl>`{lang="html"}](/scripts/maplibre/api/navigation-control) 添加缩放、指南针和俯仰控制。
- [`<ScriptMapLibreGeoJson>`{lang="html"}](/scripts/maplibre/api/geojson) 管理 GeoJSON 源及其样式图层。

## 指南

- [配送追踪器](/scripts/maplibre/guides/delivery-tracker) 使用响应式标记和样式化 GeoJSON 构建实用的路线追踪器。
- [样式与提供商](/scripts/maplibre/guides/styles-and-providers) 区分渲染器、样式、瓦片服务和地图数据的选择。
- [性能、CSP 与无障碍](/scripts/maplibre/guides/performance-csp-and-accessibility) 涵盖加载触发器、WebGL 回退方案、工作线程和装饰性地图。
