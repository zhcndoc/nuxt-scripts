---
title: Calendly
description: 在你的 Nuxt 应用中嵌入 Calendly 预约，支持内联、弹窗和徽章小组件。
links:
  - label: useScriptCalendly
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/calendly.ts
    size: xs
  - label: "<ScriptCalendlyInlineWidget>"
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptCalendlyInlineWidget.vue
    size: xs
---

[Calendly](https://calendly.com) 提供预约页面和可嵌入的日程安排小组件。其预约流程可以以内联方式、弹窗方式或浮动徽章按钮的形式呈现。

使用 [`useScriptCalendly()`{lang="ts"}](/scripts/calendly) 来实现弹窗和徽章小组件。对于内联预约流程，请使用无头组件 [`<ScriptCalendlyInlineWidget>`{lang="html"}](/scripts/calendly){lang="html"}。

::script-stats
::

::script-docs
::

生成的架构还会列出 `url`、`prefill`、`utm` 和 `pageSettings`，但当前的组合式函数不会将这些顶层值应用于小组件。请按照下方示例将它们传递给初始化器，或使用 `<ScriptCalendlyInlineWidget>`{lang="html"}。

默认值：

- **触发器：`onNuxtReady`** 直接调用组合式函数会在 Nuxt 应用准备就绪时加载脚本。内联组件则使用元素触发器。
- **样式表：内联** 小组件的样式表（以及其关闭图标 SVG）会在首次使用时以内联方式加载，因此渲染小组件不会从 `assets.calendly.com` 请求这些资源。

资源代理仅覆盖 `assets.calendly.com`。预约 iframe 仍会直接连接到 `calendly.com`。

对于无返回值的调用，请使用代理。需要已加载的 `Calendly` 全局对象或稳定的 DOM 引用时，请使用 `onLoaded`。

::code-group

```ts [代理]
const { proxy } = useScriptCalendly()
function openBooking() {
  proxy.Calendly.initPopupWidget({
    url: 'https://calendly.com/your-name/30min',
  })
}
```

```ts [加载完成]
const { onLoaded } = useScriptCalendly()
onLoaded(({ Calendly }) => {
  Calendly.initInlineWidget({
    url: 'https://calendly.com/your-name/30min',
    parentElement: document.getElementById('calendly-inline')!,
  })
})
```

::】【。

## [`<ScriptCalendlyInlineWidget>`{lang="html"}](/scripts/calendly){lang="html"}

该组件会在你控制的宿主元素中挂载 Calendly 的内联预约流程。

它会等待宿主元素进入视口后再加载组件脚本。默认的[元素触发器](/docs/guides/script-triggers#element-event-triggers)是 `'visible'`。

```vue
<script setup lang="ts">
const ready = ref(false)
</script>

<template>
  <ScriptCalendlyInlineWidget
    url="https://calendly.com/your-name/30min"
    @ready="ready = true"
  />
</template>
```

### 首屏加载

对于首屏可见的组件，请设置 `above-the-fold` 并将触发器切换为 hydration。这还会为 `calendly.com` 添加预连接。

```vue
<ScriptCalendlyInlineWidget
  url="https://calendly.com/your-name/30min"
  above-the-fold
  trigger="onNuxtReady"
/>
```

### 预填充、UTM 和页面设置

```vue
<ScriptCalendlyInlineWidget
  url="https://calendly.com/your-name/30min"
  :prefill="{ name: 'Ada Lovelace', email: 'ada@example.com' }"
  :utm="{ utmSource: 'website', utmMedium: 'cta', utmCampaign: 'launch' }"
  :page-settings="{ hideEventTypeDetails: true, hideGdprBanner: true }"
/>
```

### 插槽

该组件提供 `loading`、`awaitingLoad` 和 `error` 插槽，用于在脚本触发器等待期间或脚本加载失败时显示占位 UI。默认的 `loading` 插槽会渲染一个无障碍的加载指示器。

## 弹窗和徽章小组件

弹窗和徽章小组件没有宿主元素。通过组合式函数打开它们：

::code-group

```ts [弹窗]
const { proxy } = useScriptCalendly()
function open() {
  proxy.Calendly.initPopupWidget({
    url: 'https://calendly.com/your-name/30min',
  })
}
```

```ts [徽章]
const { onLoaded } = useScriptCalendly()
onLoaded(({ Calendly }) => {
  Calendly.initBadgeWidget({
    url: 'https://calendly.com/your-name/30min',
    text: 'Schedule time with me',
    color: '#0069ff',
    textColor: '#ffffff',
  })
})
```

::

## 预填邀请人信息和 UTM 参数

所有四个小组件初始化器（`initInlineWidget`、`initPopupWidget`、`initBadgeWidget`、`initPopupWidgetWithText`）都接受 `prefill` 和 `utm` 选项，用于预填预约表单并为预约添加营销归因标签。Calendly 的 [UTM 指南](https://help.calendly.com/hc/en-us/articles/4406950779799?locale=en-us)介绍了 JavaScript 选项名称以及受支持的嵌入类型。

```vue
<script setup lang="ts">
const { proxy } = useScriptCalendly()

function bookFromCampaign(user: { name: string, email: string }) {
  proxy.Calendly.initPopupWidget({
    url: 'https://calendly.com/your-name/30min',
    prefill: {
      name: user.name,
      email: user.email,
    },
    utm: {
      utmSource: 'website',
      utmMedium: 'cta',
      utmCampaign: 'launch',
    },
  })
}
</script>
```

::script-types
::
