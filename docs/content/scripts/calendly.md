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

[Calendly](https://calendly.com) 是一个日程安排工具，可让访客无需反复发送邮件就能在你的日历上预约时间。Calendly 嵌入式小组件可将预约流程以内联、弹窗或浮动徽章按钮的形式呈现。

Nuxt Scripts 提供了一个注册脚本组合式函数 [`useScriptCalendly()`{lang="ts"}](/scripts/calendly) 以及一个无头的 [`<ScriptCalendlyInlineWidget>`{lang="html"}](/scripts/calendly){lang="html"} 组件，方便你将其集成到 Nuxt 应用中。

::script-stats
::

::script-docs
::

该组合式函数包含以下默认值：
- **触发时机：客户端** 脚本会在 Nuxt 进行 hydration 时加载。
- **样式表：内联** 小组件样式表（以及其关闭图标的 SVG）会在首次使用时以内联方式注入，因此渲染时不会向 `assets.calendly.com` 泄露 IP。

你可以直接通过代理访问 `Calendly` 全局对象，或者等待 `onLoaded` 后再使用它。推荐在无需返回值的调用中使用代理；当你需要一个稳定的 DOM 引用时，`onLoaded` 会更方便。

::code-group

```ts [Proxy]
const { proxy } = useScriptCalendly()
function openBooking() {
  proxy.Calendly.initPopupWidget({
    url: 'https://calendly.com/your-name/30min',
  })
}
```

```ts [onLoaded]
const { onLoaded } = useScriptCalendly()
onLoaded(({ Calendly }) => {
  Calendly.initInlineWidget({
    url: 'https://calendly.com/your-name/30min',
    parentElement: document.getElementById('calendly-inline')!,
  })
})
```

::

## [`<ScriptCalendlyInlineWidget>`{lang="html"}](/scripts/calendly){lang="html"}

[`<ScriptCalendlyInlineWidget>`{lang="html"}](/scripts/calendly){lang="html"} 组件对 [`useScriptCalendly()`{lang="ts"}](/scripts/calendly){lang="ts"} 做了封装，用于最常见的嵌入形式：将内联预约流程挂载到你控制的宿主元素中。

它通过使用 [Element Event Triggers](/docs/guides/script-triggers#element-event-triggers) 进行了性能优化，只有当宿主元素进入视口时才会加载 Calendly 小组件脚本。默认触发器为 `'visible'`。

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

如果小组件位于首屏内，并且你希望它在 hydration 时就开始加载，而不是等到可见时再加载，请设置 `above-the-fold`（这会向 `calendly.com` 添加预连接），并覆盖触发器。

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

弹窗和徽章模式没有宿主元素，因此需要直接通过组合式函数驱动：

::code-group

```ts [Popup]
const { proxy } = useScriptCalendly()
function open() {
  proxy.Calendly.initPopupWidget({
    url: 'https://calendly.com/your-name/30min',
  })
}
```

```ts [Badge]
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

四种小组件初始化器（`initInlineWidget`、`initPopupWidget`、`initBadgeWidget`、`initPopupWidgetWithText`）都接受 `prefill` 和 `utm` 选项，用于预先填写预约表单，并为该预约附加营销归因标签。

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

## 示例

当 Nuxt 准备就绪时，通过 `app.vue` 加载 Calendly，并在预约页面上渲染内联小组件。

```vue [app.vue]
<script setup lang="ts">
useScriptCalendly({
  scriptOptions: {
    trigger: 'onNuxtReady',
  },
})
</script>
```
