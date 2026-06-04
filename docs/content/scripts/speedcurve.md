---
title: SpeedCurve LUX
description: 在你的 Nuxt 应用中使用 SpeedCurve LUX Real User Monitoring 来衡量真实用户所体验到的性能，并自动跟踪 SPA 导航。
links:
  - label: Source
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/speedcurve.ts
    size: xs
---

[SpeedCurve LUX](https://speedcurve.com/features/lux/) 是一款 Real User Monitoring（RUM）工具，用于衡量用户所体验到的性能。它会跟踪 Core Web Vitals、自定义时间标记以及 JavaScript 错误。

::script-stats
::

::script-docs
::

该 composable 具有以下默认值：
<!-- eslint-disable-next-line harlanzw/ai-deslop-passive-voice -->
- **Trigger: Client** LUX primer 会立即注入到 `<head>`{lang="html"} 中；当 Nuxt 完成 hydration 时，`lux.js` 会加载。

## Setup

SpeedCurve LUX 采用按需启用。请将其注册到 `scripts.registry.speedcurve` 中，以便在构建时解析并内联 LUX primer。请同时安装 `@speedcurve/lux` peer 依赖：

```bash
pnpm add -D @speedcurve/lux
```

```ts [nuxt.config.ts: composable-only (no global load)]
export default defineNuxtConfig({
  modules: ['@nuxt/scripts'],
  scripts: {
    registry: {
      // 最小注册——为每个页面启用该 composable。
      // 在这里传入 `id`，就可以在每次调用 useScriptSpeedCurve() 时省略它。
      speedcurve: {},
    },
  },
})
```

```ts [nuxt.config.ts: auto-load globally]
export default defineNuxtConfig({
  modules: ['@nuxt/scripts'],
  scripts: {
    registry: {
      speedcurve: { id: 'YOUR_SPEEDCURVE_ID', trigger: 'onNuxtReady' },
    },
  },
})
```

如果未注册 `speedcurve`，`useScriptSpeedCurve` 会使用空的 primer 作为回退进行构建。如果已注册但缺少 `@speedcurve/lux`，构建将失败并提示安装。锁定你自己的 `@speedcurve/lux` 版本意味着你可以控制 primer 片段何时更新。

你可以直接通过代理访问 `LUX` 对象，或者等待 `$script` 来获取已加载的实例。

::code-group

```ts [Proxy]
const { proxy } = useScriptSpeedCurve({ id: 'YOUR_ID' })
proxy.LUX.label = 'my-page'
```

```ts [onLoaded]
const { onLoaded } = useScriptSpeedCurve({ id: 'YOUR_ID' })
onLoaded(({ LUX }) => {
  LUX.label = 'my-page'
})
```

::

## SPA navigation

设置 `spaMode: true` 以启用 SpeedCurve 的 SPA 跟踪模式。该 composable 会自动连接 Vue Router：

- `router.beforeEach` 调用 `LUX.startSoftNavigation()`{lang="ts"}（关闭上一个 beacon，开始新的 beacon）
- `nuxt.hook('page:finish')`{lang="ts"} 在下一次绘制后调用 `LUX.markLoadTime()`{lang="ts"}（设置 END 标记）
- 被取消的导航会通过 `addData('luxNavFailed', '1')`{lang="ts"} 封存虚假的 beacon，便于过滤

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

## Custom page labels

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

参考：https://support.speedcurve.com/docs/add-rum-to-your-csp

::script-types
::

## Example

通过 `app.vue` 加载 SpeedCurve LUX，并启用 SPA 跟踪。

```vue [app.vue]
<script setup lang="ts">
useScriptSpeedCurve({
  id: 'YOUR_SPEEDCURVE_ID',
  spaMode: true,
})
</script>
```
