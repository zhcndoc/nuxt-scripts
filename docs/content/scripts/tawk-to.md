---
title: Tawk.to
description: 加载 Tawk.to 实时聊天小组件，并通过类型化代理、响应式状态和事件监听器控制它。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/tawk-to.ts
    size: xs
---

[Tawk.to](https://www.tawk.to/) 是一个免费的实时聊天小组件。

[`useScriptTawkTo()`{lang="ts"}](/scripts/tawk-to) 会加载小组件、为 `window.Tawk_API` 命令接口提供类型定义，并将嵌入脚本分发的 `window` 事件桥接到响应式状态和类型化监听器。

::script-stats
::

::script-docs
::

打包和代理功能均已关闭。尚无人验证嵌入脚本如何解析其自身的 API 来源，也不清楚代理最终会代理哪些实时聊天轮询连接，因此这两项能力都未予声明，而不是凭猜测启用。

在 Tawk.to 控制面板的 **Administration Settings → Channels → Chat Widget** 下查找 `propertyId` 和 `widgetId`。

::code-group

```ts [Proxy]
const { proxy } = useScriptTawkTo({
  propertyId: 'YOUR_PROPERTY_ID',
  widgetId: 'YOUR_WIDGET_ID',
})

function openChat() {
  proxy.maximize()
}
```

```ts [onLoaded]
const { onLoaded } = useScriptTawkTo({
  propertyId: 'YOUR_PROPERTY_ID',
  widgetId: 'YOUR_WIDGET_ID',
})

onLoaded((Tawk_API) => {
  Tawk_API.maximize()
})
```

::

## 响应式状态和事件

Tawk 的嵌入脚本会分发 `window` `CustomEvent`（`tawkLoad`、`tawkStatusChange`、`tawkChatMaximized`、……），同时还提供文档中所述的 `Tawk_API.onXxx = fn` 回调属性 API。`useScriptTawkTo()`{lang="ts"} 会将这些事件桥接到五个只读 ref 和二十个类型化监听器，因此你无需自行配置 `window.addEventListener`：

```vue
<script setup lang="ts">
const { isHidden, isMinimized, isMaximized, chatStatus, unreadCount, onChatStarted, onChatEnded } = useScriptTawkTo({
  propertyId: 'YOUR_PROPERTY_ID',
  widgetId: 'YOUR_WIDGET_ID',
})

onChatStarted(() => {
  console.log('visitor started a chat')
})
onChatEnded(() => {
  console.log('chat ended')
})
</script>

<template>
  <div v-if="!isHidden">
    {{ chatStatus }} · {{ unreadCount }} unread · {{ isMinimized ? 'minimized' : isMaximized ? 'maximized' : 'default' }}
  </div>
</template>
```

`chatStatus` 是 Tawk 自身的在线／离线／暂离客服状态（`getStatus()`{lang="ts"}）。它不同于通用的脚本加载状态 `status`，每个 registry 条目都会提供该状态。

每个 `onXxx` 监听器都会返回一个清理函数，供 `onScopeDispose` 使用，与 registry 中其他事件监听器辅助函数的行为一致。

状态 ref 是页面上所有 `useScriptTawkTo()`{lang="ts"} 调用共享的单个实例（页面上始终只有一个 Tawk 小组件），而不是每次调用各自拥有一个实例。

## Getter

`proxy` 采用即发即弃模式：脚本加载前的调用会排队，并在脚本加载后重放，但其返回值始终会被丢弃，即使脚本已经加载也是如此。对于 `proxy.maximize()`{lang="ts"} 这类本来也不返回任何有意义内容的操作来说，这没有问题，但它无法传递真正的同步 getter。`getWindowType`、`getStatus`、`isChatMaximized`、`isChatMinimized`、`isChatHidden`、`isChatOngoing`、`isVisitorEngaged` 和 `widgetPosition` 会直接暴露在 `useScriptTawkTo()`{lang="ts"} 的返回值中，并直接调用 `window.Tawk_API`：

```ts
const { getStatus, isChatHidden } = useScriptTawkTo({
  propertyId: 'YOUR_PROPERTY_ID',
  widgetId: 'YOUR_WIDGET_ID',
})

getStatus() // 'online' | 'away' | 'offline' | undefined
isChatHidden() // boolean, false before the widget has loaded
```

## 识别访客

出于同样的原因，`proxy.visitor = {...}` 不起作用：unhead 的脚本代理没有 `set` 陷阱，因此通过它进行的属性赋值无法传递到真正的 `Tawk_API`。请改用 `setVisitor()`{lang="ts"}。

该方法只能在加载前调用。Tawk 会在嵌入脚本加载前接受 `Tawk_API.visitor`，加载后则会忽略它，因此嵌入脚本请求发出后再调用会发出警告并且不执行任何操作。加载后若要更改身份，请使用 `window.Tawk_API.setAttributes({ name, email, phone, hash })`{lang="ts"}。

```ts
const { proxy, setVisitor } = useScriptTawkTo({
  propertyId: 'YOUR_PROPERTY_ID',
  widgetId: 'YOUR_WIDGET_ID',
})

setVisitor({
  name: 'Jane Doe',
  email: 'jane@example.com',
  // HMAC-SHA256 signature for Secure Mode, generated server-side
  hash: visitorHash,
})
proxy.setAttributes({ plan: 'pro' })
proxy.addTags(['vip'])
```

## 在运行时切换属性

```ts
const { proxy } = useScriptTawkTo({
  propertyId: 'YOUR_PROPERTY_ID',
  widgetId: 'YOUR_WIDGET_ID',
})

proxy.switchWidget({ propertyId: 'OTHER_PROPERTY_ID', widgetId: 'OTHER_WIDGET_ID' })
```

::script-types
::

## Partytown

不要在 Partytown 下运行 Tawk.to。该小组件会直接渲染 DOM 覆盖层（聊天气泡、聊天前面板和完整聊天面板），而响应式状态和监听器所依赖的 `window` `CustomEvent` 未配置为转发到 worker。
