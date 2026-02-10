---
title: 谷歌地图
description: 在你的 Nuxt 应用中展示性能优化的谷歌地图。
links:
  - label: useScriptGoogleMaps
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-maps.ts
    size: xs
  - label: "<ScriptGoogleMaps>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/components/ScriptGoogleMaps.vue
    size: xs
---

[谷歌地图](https://maps.google.com/) 允许你将地图嵌入你的网站并用自己的内容进行自定义。

Nuxt Scripts 提供了一个 `useScriptGoogleMaps` 组合式函数和一个无 UI 的 `ScriptGoogleMaps` 组件，用于与谷歌地图交互。

## 类型

要使用带有完整 TypeScript 支持的谷歌地图，你需要安装 `@types/google.maps` 依赖。

```bash
pnpm add -D @types/google.maps
```

## ScriptGoogleMaps

`ScriptGoogleMaps` 组件是对 `useScriptGoogleMaps` 组合式函数的封装。它提供了一种简单方式在你的 Nuxt 应用中嵌入谷歌地图。

它通过利用[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)进行了性能优化，仅在特定元素事件发生时加载谷歌地图。

在谷歌地图加载之前，它会使用[静态地图 API](https://developers.google.com/maps/documentation/maps-static)显示一个占位图。

默认情况下，地图会在 `mouseover` 和 `mouseclick` 事件触发时加载。

### 计费与权限

::callout
你需要一个拥有访问[地图 JavaScript API](https://developers.google.com/maps/documentation/javascript/cloud-setup)权限的 API 密钥。

可选地，你可以提供对[静态地图 API](https://developers.google.com/maps/documentation/maps-static/cloud-setup)（使用懒加载和占位地图时必需）和[地点 API](https://developers.google.com/maps/documentation/places/web-service/cloud-setup)（使用查询搜索时，例如“纽约”）的权限。
::

显示交互式 JS 地图需要地图 JavaScript API，它是一个付费服务。如果用户与地图交互，将产生以下费用：
- 地图 JavaScript API：每 1000 次加载 7 美元（默认使用谷歌地图）
- 静态地图 API：每 1000 次加载 2 美元——仅当你没有提供 `placeholder` 插槽时使用
- 地理编码 API：每 1000 次加载 5 美元——仅当你没有为 `center` 属性提供 `google.maps.LatLng` 对象而是使用查询字符串时使用

但是，如果用户从未与地图互动，且你使用了静态地图 API，则只会产生静态地图 API 的费用（每 1000 次加载 2 美元）。

计费将在[未来更新](https://github.com/nuxt/scripts/issues/83)中优化。

如果你想避免这些费用，并且能接受功能较弱的地图，可以考虑改用[iframe 嵌入](https://developers.google.com/maps/documentation/embed/get-started)。

### 演示

::code-group

:google-maps-demo{label="输出"}

```vue [输入]
<script setup lang="ts">
import { ref } from 'vue'

const isLoaded = ref(false)
const center = ref()
const maps = ref()

const query = ref({
  lat:  -37.7995487,
  lng: 144.9867841,
})

const markers = ref([])

let increment = 1
function addMarker() {
  // 向 markers 中添加标记，我们希望从中心位置添加但随机稍微偏移
  const _center = center.value || query.value
  // lat 和 lng 可能是函数
  const _lat = typeof _center.lat === 'function' ? _center.lat() : _center.lat
  const _lng = typeof _center.lng === 'function' ? _center.lng() : _center.lng
  const lat = (1000 * _lat + increment) / 1000
  const lng = (1000 * _lng + increment) / 1000
  increment += 1

  markers.value.push(`${lat},${lng}`)
}

function removeMarkers() {
  markers.value = []
  increment = 1
}
function handleReady({ map }) {
  center.value = map.value.getCenter()
  map.value.addListener('center_changed', () => {
    center.value = map.value.getCenter()
  })
  isLoaded.value = true
}
</script>

<template>
<div class="not-prose">
  <div class="flex items-center justify-center p-5">
    <ScriptGoogleMaps
      ref="maps"
      :center="query"
      :markers="markers"
      api-key="AIzaSyAOEIQ_xOdLx2dNwnFMzyJoswwvPCTcGzU"
      class="group"
      above-the-fold
      @ready="handleReady"
    />
  </div>
  <div class="text-center">
    <UAlert v-if="!isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="静态图像：移动鼠标加载交互地图" description="鼠标悬停地图时将触发谷歌地图脚本加载并初始化地图。" />
    <UAlert v-if="isLoaded" class="mb-5" size="sm" color="blue" variant="soft" title="交互式地图">
      <template #description>
      中心：{{ center }}
      </template>
    </UAlert>
    <UButton @click="addMarker" type="button" class="">
      添加标记
    </UButton>
    <UButton v-if="markers.length" @click="removeMarkers" type="button" color="gray" variant="ghost" class="">
      移除标记
    </UButton>
  </div>
</div>
</template>
```

::

### 属性

`ScriptGoogleMaps` 组件支持以下属性。

你必须提供 `center` 属性以正确加载地图，或者提供 `mapOptions` 并在其中配置 `center` 选项。

**地图**

- `center`：地图中心所在的位置。你可以提供一个位置字符串或 `{ lat: 0, lng: 0 }` 对象。
- `apiKey`：谷歌地图 API 密钥，必须也有访问静态地图 API 的权限。你也可以通过运行时配置 `public.scripts.googleMaps.apiKey` 提供。
- `centerMarker`：是否在中心位置显示标记。默认值为 `true`。
- `mapOptions`：地图选项。详见 [MapOptions](https://developers.google.com/maps/documentation/javascript/reference/map#MapOptions)。

**占位符**

你可以通过以下属性自定义占位图，或者使用 `#placeholder` 插槽自定义占位图。

- `placeholderOptions`：自定义占位图的属性。详见 [静态地图 API](https://developers.google.com/maps/documentation/maps-static/start)。
- `placeholderAttrs`：自定义占位图的 HTML 属性。

**尺寸**

如果你想渲染大于 640x640 的地图，应自行提供占位图，因为[静态地图 API](https://developers.google.com/maps/documentation/maps-static/start)不支持渲染超过这个尺寸的地图。

- `width`：地图宽度，默认 `640`。
- `height`：地图高度，默认 `400`。

**优化**

- `trigger`：加载谷歌地图的触发事件，默认是 `mouseover`。更多信息见[元素事件触发器](/docs/guides/script-triggers#element-event-triggers)。
- `aboveTheFold`：针对首屏内容优化占位图，默认 `false`。

**标记**

你可以通过提供标记数组添加标记，数组内元素类型为 `MarkerOptions`。详见[MarkerOptions](https://developers.google.com/maps/documentation/javascript/reference/marker#MarkerOptions)。

- `markers`：用于地图上显示的标记数组。

详细信息请参考 [markers 示例](https://github.com/nuxt/scripts/blob/main/playground/pages/third-parties/google-maps/markers.vue)。

#### 使用环境变量

如果你希望通过环境变量配置 API 密钥。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleMaps: true,
    }
  },
  // 需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        googleMaps: {
          apiKey: '', // NUXT_PUBLIC_SCRIPTS_GOOGLE_MAPS_API_KEY
        },
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_GOOGLE_MAPS_API_KEY=<你的API密钥>
```

### 指南

#### 预加载占位图

谷歌地图的占位图默认采用懒加载。如果你的地图位于首屏位置，建议修改此行为或使用 `#placeholder` 插槽自定义占位图。

::code-group

```vue [Placeholder Attrs]
<ScriptGoogleMaps above-the-fold />
```

```vue [Placeholder Slot]
<ScriptGoogleMaps>
  <template #placeholder="{ placeholder }">
    <img :src="placeholder" alt="地图占位图">
  </template>
</ScriptGoogleMaps>
```

::

#### 高级标记控制

如果你需要对地图标记有更高级的控制，可以使用暴露的 `createAdvancedMapMarker` 方法，它会返回标记实例。

```vue
<script lang="ts" setup>
const googleMapsRef = ref()
onMounted(() => {
  const marker = googleMapsRef.value.createAdvancedMapMarker({
    position: { }
  })
})
</script>
<template>
    <ScriptGoogleMaps ref="googleMapsRef" />
</template>
```

#### 高级地图控制

该组件暴露了所有内部 API，你可以根据需要进行地图定制。

```vue
<script lang="ts" setup>
const googleMapsRef = ref()
onMounted(async () => {
  const api = googleMapsRef.value
  
  // 访问内部 API
  const googleMaps = api.googleMaps.value // google.maps api
  const mapInstance = api.map.value // google.maps.Map 实例
  
  // 将查询转换为经纬度
  const query = await api.resolveQueryToLatLang('Space Needle, Seattle, WA') // { lat: 0, lng: 0 }
  
  // 导入谷歌地图库
  const geometry = await api.importLibrary('geometry')
  const distance = new googleMaps.geometry.spherical.computeDistanceBetween(
    new googleMaps.LatLng(0, 0),
    new googleMaps.LatLng(0, 0)
  )
})
</script>
<template>
    <ScriptGoogleMaps ref="googleMapsRef" />
</template>
```

#### 立即加载

如果你想立即加载谷歌地图，可以使用 `trigger` 属性。

```vue
<template>
<ScriptGoogleMaps trigger="immediate">
</ScriptGoogleMaps>
</template>
```

#### 地图样式

你可以通过 `mapOptions.styles` 属性为地图设置样式。你可以在[Snazzy Maps](https://snazzymaps.com/)上找到预制样式。

这会同时应用于静态地图占位图和交互式地图。

```vue
<script setup lang="ts">
const mapOptions = {
  styles: [{ elementType: 'labels', stylers: [{ visibility: 'off' }, { color: '#f49f53' }] }, { featureType: 'landscape', stylers: [{ color: '#f9ddc5' }, { lightness: -7 }] }, { featureType: 'road', stylers: [{ color: '#813033' }, { lightness: 43 }] }, { featureType: 'poi.business', stylers: [{ color: '#645c20' }, { lightness: 38 }] }, { featureType: 'water', stylers: [{ color: '#1994bf' }, { saturation: -69 }, { gamma: 0.99 }, { lightness: 43 }] }, { featureType: 'road.local', elementType: 'geometry.fill', stylers: [{ color: '#f19f53' }, { weight: 1.3 }, { visibility: 'on' }, { lightness: 16 }] }, { featureType: 'poi.business' }, { featureType: 'poi.park', stylers: [{ color: '#645c20' }, { lightness: 39 }] }, { featureType: 'poi.school', stylers: [{ color: '#a95521' }, { lightness: 35 }] }, {}, { featureType: 'poi.medical', elementType: 'geometry.fill', stylers: [{ color: '#813033' }, { lightness: 38 }, { visibility: 'off' }] },
}
</script>
<template>
<ScriptGoogleMaps :mapOptions="mapOptions" />
</template>
```

### 组件 API

完整的 Props、事件和插槽详情，参见[Facade 组件 API](/docs/guides/facade-components#facade-components-api)。

### 事件

`ScriptGoogleMaps` 组件在谷歌地图加载完成时触发单个 `ready` 事件。

```ts
const emits = defineEmits<{
  ready: [map: google.maps.Map]
}>()
```

你可以通过监听 `ready` 事件来订阅谷歌地图事件。

```vue
<script setup lang="ts">
function handleReady({ map }) {
  map.addListener('center_changed', () => {
    console.log('中心已改变', map.getCenter())
  })
}
</script>

<template>
  <ScriptGoogleMaps @ready="handleReady" />
</template>
```

### 插槽

组件默认只提供最小化的 UI，仅保证功能和无障碍性。你可以通过多个插槽自由定制地图显示。

**default**

默认插槽用于显示始终可见的内容。

```vue
<template>
  <ScriptGoogleMaps>
    <div class="absolute top-0 left-0 right-0 p-5 bg-white text-black">
      <h1 class="text-xl font-bold">
        我的自定义地图
      </h1>
    </div>
  </ScriptGoogleMaps>
</template>
```

**awaitingLoad**

该插槽用于谷歌地图加载过程中的显示内容。

```vue
<template>
  <ScriptGoogleMaps>
    <template #awaitingLoad>
      <div class="bg-blue-500 text-white p-5">
        点击加载地图！
      </div>
    </template>
  </ScriptGoogleMaps>
</template>
```

**loading**

该插槽用于谷歌地图加载时的显示内容。

注意：默认会显示一个 `ScriptLoadingIndicator` 以提升无障碍性和用户体验，若自定义此插槽会覆盖该默认组件，请确保提供加载指示器。

```vue
<template>
  <ScriptGoogleMaps>
    <template #loading>
      <div class="bg-blue-500 text-white p-5">
        加载中...
      </div>
    </template>
  </ScriptGoogleMaps>
</template>
```

**placeholder**

该插槽用于谷歌地图加载前显示占位图。默认情况下，显示谷歌地图静态 API 的图片。

若你提供了自定义占位图槽，将禁用默认占位图，且不会产生静态地图 API 的费用。

```vue
<template>
  <ScriptGoogleMaps>
    <template #placeholder="{ placeholder }">
      <img :src="placeholder">
    </template>
  </ScriptGoogleMaps>
</template>
```

## 谷歌地图单文件组件（SFC）

Nuxt Scripts 提供了多个独立的单文件组件（SFC），用于不同的谷歌地图元素。这些组件允许你通过 Vue 模板语法声明式地组合复杂地图。

### 安装

要使用标记聚合（marker clustering）功能，你需要安装对应的同级依赖包：

```bash
npm install @googlemaps/markerclusterer
# 或者
yarn add @googlemaps/markerclusterer
# 或者
pnpm add @googlemaps/markerclusterer
```

### 可用组件

所有谷歌地图 SFC 组件必须嵌套在 `<ScriptGoogleMaps>` 组件内使用：

- `<ScriptGoogleMapsMarker>` - 经典标记，支持图标
- `<ScriptGoogleMapsAdvancedMarkerElement>` - 现代高级标记，支持 HTML 内容
- `<ScriptGoogleMapsPinElement>` - 可定制的针型标记（用于高级标记中）
- `<ScriptGoogleMapsInfoWindow>` - 点击时显示的信息窗口
- `<ScriptGoogleMapsMarkerClusterer>` - 将附近标记聚合成群组
- `<ScriptGoogleMapsCircle>` - 圆形覆盖层
- `<ScriptGoogleMapsPolygon>` - 多边形形状
- `<ScriptGoogleMapsPolyline>` - 线条路径
- `<ScriptGoogleMapsRectangle>` - 矩形覆盖层
- `<ScriptGoogleMapsHeatmapLayer>` - 热力图可视化层

### 基本用法

```vue
<template>
  <ScriptGoogleMaps
    :center="{ lat: -34.397, lng: 150.644 }"
    :zoom="8"
    api-key="your-api-key"
  >
    <!-- 添加标记 -->
    <ScriptGoogleMapsMarker
      :options="{ position: { lat: -34.397, lng: 150.644 } }"
    >
      <!-- 点击标记显示信息窗口 -->
      <ScriptGoogleMapsInfoWindow>
        <div>
          <h3>澳大利亚，悉尼</h3>
          <p>一座伟大的城市！</p>
        </div>
      </ScriptGoogleMapsInfoWindow>
    </ScriptGoogleMapsMarker>

    <!-- 高级标记，带自定义针 -->
    <ScriptGoogleMapsAdvancedMarkerElement
      :options="{ position: { lat: -34.407, lng: 150.654 } }"
    >
      <ScriptGoogleMapsPinElement
        :options="{ scale: 1.5, background: '#FF0000' }"
      />
    </ScriptGoogleMapsAdvancedMarkerElement>

    <!-- 圆形覆盖层 -->
    <ScriptGoogleMapsCircle
      :options="{
        center: { lat: -34.397, lng: 150.644 },
        radius: 1000,
        fillColor: '#FF0000',
        fillOpacity: 0.35
      }"
    />
  </ScriptGoogleMaps>
</template>
```

### 组件组合示例

**标记聚合**

```vue
<template>
  <ScriptGoogleMaps api-key="your-api-key">
    <ScriptGoogleMapsMarkerClusterer>
      <ScriptGoogleMapsMarker
        v-for="location in locations"
        :key="location.id"
        :options="{ position: location.position }"
      >
        <ScriptGoogleMapsInfoWindow>
          <div>{{ location.name }}</div>
        </ScriptGoogleMapsInfoWindow>
      </ScriptGoogleMapsMarker>
    </ScriptGoogleMapsMarkerClusterer>
  </ScriptGoogleMaps>
</template>
```

**热力图与数据点**

```vue
<script setup>
const heatmapData = ref([])

onMounted(() => {
  // 用 google.maps.LatLng 对象填充热力图数据
})
</script>

<template>
  <ScriptGoogleMaps api-key="your-api-key">
    <ScriptGoogleMapsHeatmapLayer
      :options="{ data: heatmapData }"
    />
  </ScriptGoogleMaps>
</template>
```

**完整示例请参阅[SFC Playground 示例](https://nuxt-scripts-playground.stackblitz.io/third-parties/google-maps/sfcs)。**

### 组件详情

#### ScriptGoogleMapsMarker

经典谷歌地图标记，支持图标。

**属性:**
- `options` - `google.maps.MarkerOptions`（不包括 `map`）

**事件:**
- 标准标记事件：`click`、`mousedown`、`mouseover` 等。

#### ScriptGoogleMapsAdvancedMarkerElement

现代高级标记，支持 HTML 内容和更好的自定义。

**属性:**
- `options` - `google.maps.marker.AdvancedMarkerElementOptions`（不包括 `map`）

**事件:**
- 标准标记事件：`click`、`drag`、`position_changed` 等。

#### ScriptGoogleMapsInfoWindow

显示内容的窗口。

**属性:**
- `options` - `google.maps.InfoWindowOptions`

**行为:**
- 会在父级标记点击时自动打开
- 也可作为独立组件使用，并显式定义位置
- 支持通过默认插槽传入自定义 HTML 内容

#### ScriptGoogleMapsMarkerClusterer

将相近标记聚合，提升性能和用户体验。

**属性:**
- `options` - `MarkerClustererOptions`（不包括 `map`）

**依赖:**
- 需要安装 `@googlemaps/markerclusterer` 同级依赖

#### 其他组件

- **ScriptGoogleMapsPinElement**：用于高级标记的可定制针
- **ScriptGoogleMapsCircle**：带半径和样式的圆形覆盖层
- **ScriptGoogleMapsPolygon/Polyline**：形状和线条覆盖层
- **ScriptGoogleMapsRectangle**：矩形覆盖层
- **ScriptGoogleMapsHeatmapLayer**：数据可视化热力图

所有组件支持：
- 响应式 `options` 属性，自动更新谷歌地图对象
- 组件卸载时自动清理
- TypeScript 类型支持

### 最佳实践

#### 性能建议

**大量标记时使用 MarkerClusterer**
```vue
<!-- ✅ 推荐：超过 10 个标记使用聚合器 -->
<ScriptGoogleMapsMarkerClusterer>
  <ScriptGoogleMapsMarker v-for="marker in manyMarkers" />
</ScriptGoogleMapsMarkerClusterer>

<!-- ❌ 避免：大量独立标记 -->
<ScriptGoogleMapsMarker v-for="marker in manyMarkers" />
```

**现代应用推荐使用 AdvancedMarkerElement**
```vue
<!-- ✅ 推荐：性能与样式更佳 -->
<ScriptGoogleMapsAdvancedMarkerElement :options="options">
  <ScriptGoogleMapsPinElement :options="{ background: '#FF0000' }" />
</ScriptGoogleMapsAdvancedMarkerElement>

<!-- ⚠️ 旧版：仅在不支持高级标记时使用 -->
<ScriptGoogleMapsMarker :options="options" />
```

#### 组件层级关系

组件应遵循如下嵌套结构：

```
ScriptGoogleMaps (根组件)
├── ScriptGoogleMapsMarkerClusterer (可选)
│   └── ScriptGoogleMapsMarker/AdvancedMarkerElement
│       └── ScriptGoogleMapsInfoWindow (可选)
├── ScriptGoogleMapsAdvancedMarkerElement
│   ├── ScriptGoogleMapsPinElement (可选)
│   └── ScriptGoogleMapsInfoWindow (可选)
└── 其他覆盖层（Circle、Polygon 等）
```

#### 响应式数据示例

**响应式标记更新**
```vue
<script setup>
const markers = ref([
  { id: 1, position: { lat: -34.397, lng: 150.644 }, title: '悉尼' }
])

// 数据变更时标记自动更新
function addMarker() {
  markers.value.push({
    id: Date.now(),
    position: getRandomPosition(),
    title: '新地点'
  })
}
</script>

<template>
  <ScriptGoogleMaps>
    <ScriptGoogleMapsMarker
      v-for="marker in markers"
      :key="marker.id"
      :options="{ position: marker.position, title: marker.title }"
    />
  </ScriptGoogleMaps>
</template>
```

#### 错误处理

始终提供错误回退和加载状态：

```vue
<script setup>
const mapError = ref(false)
</script>

<template>
  <ScriptGoogleMaps
    @error="mapError = true"
    api-key="your-api-key"
  >
    <template #error>
      <div class="p-4 bg-red-100">
        谷歌地图加载失败
      </div>
    </template>

    <!-- 你的组件 -->
  </ScriptGoogleMaps>
</template>
```

## useScriptGoogleMaps

`useScriptGoogleMaps` 组合式函数让你可以灵活控制谷歌地图 SDK。它提供了一种方式来加载谷歌地图 SDK 并以编程方式与之交互。

```ts
export function useScriptGoogleMaps<T extends GoogleMapsApi>(_options?: GoogleMapsInput) {}
```

请参阅[Registry 脚本](/docs/guides/registry-scripts)指南了解更高级用法。

### GoogleMapsApi

```ts
export interface GoogleMapsApi {
  // @types/google.maps
  maps: typeof google.maps
}
```

## 示例

加载谷歌地图 SDK 并以编程方式操作它。

```vue
<script setup lang="ts">
/// <reference types="google.maps" />
const { onLoaded } = useScriptGoogleMaps({
  apiKey: 'key'
})
const map = ref()
onMounted(() => {
  onLoaded(async (instance) => {
    const maps = await instance.maps as any as typeof google.maps // 上游的 google 类型问题
    new maps.Map(map.value, {
      center: { lat: -34.397, lng: 150.644 },
      zoom: 8
    })
    // 可执行进一步操控
  })
})
</script>
<template>
    <div ref="map" />
</template>
```