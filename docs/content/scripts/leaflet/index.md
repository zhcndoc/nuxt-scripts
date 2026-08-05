---
title: Leaflet
description: 通过延迟加载，为 Nuxt 添加轻量、与服务提供商无关的交互式地图。
links:
  - label: useScriptLeaflet
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/leaflet.ts
    size: xs
  - label: "<ScriptLeafletMap>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/Leaflet/ScriptLeafletMap.vue
    size: xs
---

[Leaflet](https://leafletjs.com/) 是一个用于交互式地图的开源 JavaScript 库。它提供地图渲染器和交互模型；图块提供商、署名信息、地理编码和路径规划服务则由你分别选择。

Nuxt Scripts 通过 [`useScriptLeaflet()`{lang="ts"}](/scripts/leaflet/api/use-script-leaflet) 可组合函数和一小组声明式组件支持 Leaflet 1.9.4。官方模式默认捆绑 SDK，而 Nuxt Scripts 会在本地嵌入样式和标记图像。

::script-types{exclude-components}
::

## 设置

安装 Leaflet 的类型，然后启用注册表条目：

```bash
pnpm add -D @types/leaflet
```

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      leaflet: { trigger: false },
    },
  },
})
```

## 快速开始

Leaflet 不包含地图影像。此示例明确选择 OpenStreetMap 的标准瓦片服务，并保留其必需的可见署名。

```vue
<script setup lang="ts">
const center = ref<[number, number]>([-37.8136, 144.9631])
</script>

<template>
  <ScriptLeafletMap
    :center="center"
    :zoom="13"
    width="100%"
    :height="480"
    aria-label="Map of central Melbourne"
  >
    <ScriptLeafletTileLayer
      url="https://tile.openstreetmap.org/{z}/{x}/{y}.png"
      :options="{
        maxZoom: 19,
        attribution: '&copy; <a href=&quot;https://www.openstreetmap.org/copyright&quot;>OpenStreetMap contributors</a>',
      }"
    />

    <ScriptLeafletMarker
      :position="center"
      alt="Melbourne CBD"
    >
      <ScriptLeafletPopup>
        Melbourne CBD
      </ScriptLeafletPopup>
    </ScriptLeafletMarker>
  </ScriptLeafletMap>
</template>
```

默认的 `visible` 触发器会延迟加载 Leaflet SDK，并推迟所有瓦片请求，直到地图接近视口。该组件会在 SSR 期间保留其尺寸，以避免布局偏移。

::callout{color="amber"}
OpenStreetMap 的公共瓦片服务有使用限制，并且不提供 SLA。在生产环境中选择它之前，请阅读 [瓦片与署名](/scripts/leaflet/guides/tiles-and-attribution)。
::

::callout{icon="i-lucide-map-pin"}
**真实案例：** 使用[构建取货定位器](/scripts/leaflet/guides/pickup-locator)，实现支持键盘访问的门店列表、响应式选择、标记弹出窗口、错误回退内容以及 GeoJSON 配送区域。
::

## 组件

- [`<ScriptLeafletMap>`{lang="html"}](/scripts/leaflet/api/script-leaflet-map) 创建地图并控制延迟加载。
- [`<ScriptLeafletTileLayer>`{lang="html"}](/scripts/leaflet/api/tile-layer) 连接明确的瓦片提供商。
- [`<ScriptLeafletMarker>`{lang="html"}](/scripts/leaflet/api/marker) 添加可访问、响应式的标记。
- [`<ScriptLeafletPopup>`{lang="html"}](/scripts/leaflet/api/popup) 将插槽中的 HTML 绑定到标记或坐标。
- [`<ScriptLeafletGeoJson>`{lang="html"}](/scripts/leaflet/api/geojson) 渲染内联 GeoJSON。

## 指南

- [取货点定位器](/scripts/leaflet/guides/pickup-locator) 使用标记、弹出窗口和 GeoJSON 构建实用的定位器。
- [图块与署名](/scripts/leaflet/guides/tiles-and-attribution) 解释地图影像的来源，以及生产环境部署需要考虑的事项。
- [性能与无障碍](/scripts/leaflet/guides/performance-and-accessibility) 涵盖加载触发器、SSR 尺寸设置、备用内容和装饰性地图。
