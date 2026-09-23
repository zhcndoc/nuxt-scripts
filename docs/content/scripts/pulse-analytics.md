---
title: Pulse Analytics
description: 加载 Pulse 跟踪器并记录自定义事件。
links:
  - label: 来源
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/pulse-analytics.ts
    size: xs
---

[Pulse](https://pulse.ciphera.net/) 是由 [Ciphera](https://ciphera.net/) 提供的无 Cookie 网站分析服务。跟踪器不会在浏览器中留下任何内容，没有 Cookie，也没有存储的标识符；Pulse 会在自己的服务器上通过轮换哈希来识别访客。它无需额外配置即可遵守 Do Not Track 和 Global Privacy Control。[脚本参考](https://docs.ciphera.net/pulse/script-installation)列出了此组合式函数映射的所有属性。

::script-stats
::

::script-docs
::

## 不支持代理

Pulse 会根据连接的 IP 地址和用户代理，在服务器上构建访客身份。将信标请求路由到 Nuxt 服务器后，所有请求都会来自同一个 IP，因此使用相同用户代理的访客都会被合并为同一个身份。其[机器人过滤](https://docs.ciphera.net/pulse/bot-filtering)也会将数据中心或托管服务提供商的来源作为一项判断信号，而代理服务器正是位于这类来源中。

因此，Nuxt Scripts 会打包跟踪器并从你的来源提供服务，但跟踪器发出的信标请求会直接发送到 Pulse API。客户端不会因此丢失任何信息：跟踪器不会在浏览器中保存标识符，因此也无需通过第一方代理进行保护。

## 自托管或代理 API

`apiUrl` 用于设置跟踪器要向其发送请求的来源。留空则使用托管的 Pulse API。

```ts
useScriptPulseAnalytics({
  domain: 'YOUR_DOMAIN',
  apiUrl: 'https://pulse-api.example.com',
})
```

## 自定义事件

使用组合式函数的 `proxy` 对象调用 `track`。如果在脚本加载前调用，Nuxt Scripts 会暂存该调用，并在跟踪器加载后重放。如果访客选择退出，跟踪器将不会运行，队列也会被丢弃。

::code-group

```ts [Proxy]
const { proxy } = useScriptPulseAnalytics()
function trackSignup() {
  proxy.track('signup', { plan: 'pro' })
}
```

```ts [onLoaded]
const { onLoaded } = useScriptPulseAnalytics()
onLoaded(({ track }) => {
  track('purchase', { product: 'annual_plan' }, 99)
})
```

::

事件名称可以包含字母、数字和下划线。属性值必须是字符串，可选的第三个参数用于表示收入。请参阅[自定义事件参考](https://docs.ciphera.net/pulse/custom-events)。

::script-types
::

## 示例

默认触发器会等待 Nuxt 就绪：

```vue [app.vue]
<script setup lang="ts">
useScriptPulseAnalytics({
  domain: 'YOUR_DOMAIN',
})
</script>
```
