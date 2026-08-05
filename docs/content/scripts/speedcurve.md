---
title: SpeedCurve LUX
description: 在你的 Nuxt 应用中使用 SpeedCurve LUX Real User Monitoring 来衡量真实用户所体验到的性能，并自动跟踪 SPA 导航。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/speedcurve.ts
    size: xs
---

[SpeedCurve LUX](https://www.speedcurve.com/features/performance-monitoring/) 收集现场性能数据，包括核心网页指标、自定义计时标记和 JavaScript 错误。

::script-stats
::

::script-docs
::

默认情况下，该组合式函数会立即将 LUX primer 注入 `<head>`{lang="html"}，并在 Nuxt 水合期间加载 `lux.js`。

## 设置

SpeedCurve LUX 采用按需启用。请将其注册到 `scripts.registry.speedcurve` 中，以便在构建时解析并内联 LUX primer。请同时安装 `@speedcurve/lux` peer 依赖：

```bash
pnpm add -D @speedcurve/lux
```

```ts [nuxt.config.ts: 仅使用组合式函数（不全局加载）]
export default defineNuxtConfig({
  modules: ['@nuxt/scripts'],
  scripts: {
    registry: {
      // 最低限度的注册：为每个页面启用组合式函数。
      // 在此处传入 `id` 后，就可以在每次调用 useScriptSpeedCurve() 时省略它。
      speedcurve: {},
    },
  },
})
```

```ts [nuxt.config.ts: 全局自动加载]
export default defineNuxtConfig({
  modules: ['@nuxt/scripts'],
  scripts: {
    registry: {
      speedcurve: { id: 'YOUR_SPEEDCURVE_ID', trigger: 'client' },
    },
  },
})
```

对于 SpeedCurve，上面的注册表 `trigger` 是生成全局组合式函数调用的标记。`useScriptSpeedCurve()`{lang="ts"} 将有效触发器硬编码为 `client`，因此注册表或组合式函数选项中的 `onNuxtReady`、手动、同意以及自定义 Promise 触发器都不会改变其加载时机。

如果未注册 `speedcurve`，`useScriptSpeedCurve` 将使用空的 primer 回退值进行构建。如果已注册但缺少 `@speedcurve/lux`，构建将失败并提示安装。固定你自己的 `@speedcurve/lux` 版本意味着你可以控制 primer 代码片段的更新时间。

你可以直接通过代理访问 `LUX` 对象，或者等待 `$script` 来获取已加载的实例。

::code-group

```ts [代理]
const { proxy } = useScriptSpeedCurve({ id: 'YOUR_ID' })
proxy.LUX.label = 'my-page'
```

```ts [加载完成时]
const { onLoaded } = useScriptSpeedCurve({ id: 'YOUR_ID' })
onLoaded(({ LUX }) => {
  LUX.label = 'my-page'
})
```

::

## SPA 导航

设置 `spaMode: true` 以启用 SpeedCurve 的 [SPA 跟踪模式](https://support.speedcurve.com/docs/single-page-applications)。该组合式函数会自动接入 Vue Router：

- `router.beforeEach` 调用 `LUX.startSoftNavigation()`{lang="ts"}，关闭之前的 beacon 并启动新的 beacon
- `nuxt.hook('page:finish')`{lang="ts"} 会在下一次绘制后调用 `LUX.markLoadTime()`{lang="ts"}，以设置 END 标记
- 已取消的导航会通过 `addData('luxNavFailed', '1')`{lang="ts"} 为虚拟 beacon 添加标记

请将其安装在 `app.vue` 中。Nuxt Scripts 会在每个浏览器会话中创建一次自动路由钩子，因此首次自动跟踪的调用会提供其标签和导航选项。

```ts [app.vue]
useScriptSpeedCurve({
  id: 'YOUR_ID',
  spaMode: true,
  autoTrackSpaNavigations: true, // 当 spaMode 为 true 时的默认值
})
```

要禁用自动连接并手动埋点：

```ts
useScriptSpeedCurve({
  id: 'YOUR_ID',
  spaMode: true,
  autoTrackSpaNavigations: false,
})
// 然后你自己调用 LUX.startSoftNavigation() 和 LUX.markLoadTime()
```

## 自定义页面标签

默认情况下，该 composable 会使用 `String(to.name ?? to.path)`{lang="ts"} 作为每次导航的页面标签。传入一个函数到 `label` 以覆盖它：

```ts
useScriptSpeedCurve({
  id: 'YOUR_ID',
  spaMode: true,
  label: to => to.meta.title as string ?? to.path,
})
```

设置 `label: false` 可完全禁用标签。传入普通字符串可设置静态标签（仅在不使用 `spaMode` 时才有意义，因为 router hook 会在每次导航时覆盖它）。

## CSP

将以下指令添加到你的内容安全策略中：

```text
script-src  cdn.speedcurve.com;
img-src     lux.speedcurve.com;
connect-src lux.speedcurve.com beacon.speedcurve.com;
```

请参阅 SpeedCurve 的 [LUX 内容安全策略指南](https://support.speedcurve.com/docs/add-rum-to-your-csp)，了解供应商当前使用的指令。

::script-types
::

## 示例

通过 `app.vue` 加载 SpeedCurve LUX，并启用 SPA 跟踪：

```vue [app.vue]
<script setup lang="ts">
useScriptSpeedCurve({
  id: 'YOUR_SPEEDCURVE_ID',
  spaMode: true,
})
</script>
```
