---
title: Umami 分析
description: 加载 Umami、识别会话，并在事件负载发送前检查或过滤它们。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/umami-analytics.ts
    size: xs
---

[Umami](https://umami.is/) 是一个开源的网站分析平台，可以在 Umami Cloud 上运行，也可以部署在你自己的服务器上。

::script-stats
::

::script-docs
::

### 自托管的 Umami

如果你使用的是自托管版本的 Umami，请将 `hostUrl` 设置为你的 Umami 源站地址。此设置对应于 Umami 文档中的 [`data-host-url`](https://docs.umami.is/docs/tracker-configuration#data-host-url) 配置，并告知跟踪器将事件发送到何处。

```ts
useScriptUmamiAnalytics({
  websiteId: 'YOUR_WEBSITE_ID',
  hostUrl: 'https://my-self-hosted'
})
```

仅当你还需要覆盖脚本 URL 本身时，才使用 `scriptInput.src`。

## 识别会话和筛选数据

### 识别会话

Umami 的 [`identify`](https://docs.umami.is/docs/tracker-functions#sessions) 函数接受唯一 ID 或包含会话数据的对象：

```ts
const { proxy } = useScriptUmamiAnalytics({
  websiteId: 'YOUR_WEBSITE_ID'
})

// 设置唯一 ID
proxy.identify('user-12345')

// 向会话附加数据
proxy.identify({
  userId: 'user-12345',
  plan: 'premium'
})
```

### 使用 `beforeSend` 筛选数据

使用 [`beforeSend`](https://docs.umami.is/docs/tracker-configuration#data-before-send) 在 Umami 接收数据前检查、修改或取消发送的数据：

```ts
useScriptUmamiAnalytics({
  websiteId: 'YOUR_WEBSITE_ID',
  beforeSend: (type, payload) => {
    // 调试时记录正在发送的数据
    console.log('Sending to Umami:', type, payload)

    // 过滤敏感数据
    if (payload.url && payload.url.includes('private')) {
      return false // 取消发送
    }

    // 发送前修改数据
    return {
      ...payload,
      referrer: '' // 出于隐私考虑移除来源
    }
  }
})
```

你也可以提供一个全局定义函数的名字字符串：

```ts
// 全局定义函数
window.myBeforeSendHandler = (type, payload) => {
  return checkPrivacyRules(payload) ? payload : false
}

useScriptUmamiAnalytics({
  websiteId: 'YOUR_WEBSITE_ID',
  beforeSend: 'myBeforeSendHandler'
})
```

::script-types
::
