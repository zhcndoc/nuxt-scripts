---

title: Umami 分析
description: 在你的 Nuxt 应用中使用 Umami 分析。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/umami-analytics.ts
    size: xs

---

[Umami](https://umami.is/) 收集你关注的所有指标，帮助你做出更好的决策。

::script-stats
::

::script-docs
::

### 自托管的 Umami

如果你使用自托管版本的 Umami，请为脚本提供一个显式的 src，使浏览器能将 API 事件发送到正确的端点。

```ts
useScriptUmamiAnalytics({
  scriptInput: {
    src: 'https://my-self-hosted/script.js'
  }
})
```

::script-types
::

## 高级功能

### 会话识别

Umami v2.18.0 及以上版本支持使用 `identify` 函数设置唯一的会话 ID。你可以传递一个字符串（唯一 ID）或者带有会话数据的对象：

```ts
const { proxy } = useScriptUmamiAnalytics({
  websiteId: 'YOUR_WEBSITE_ID'
})

// 使用唯一字符串 ID
proxy.identify('user-12345')

// 使用会话数据对象
proxy.identify({
  userId: 'user-12345',
  plan: 'premium'
})
```

### 使用 beforeSend 进行数据过滤

`beforeSend` 选项允许你在数据发送到 Umami 之前检查、修改或取消数据。这对于实现自定义隐私控制或数据过滤非常有用：

```ts
useScriptUmamiAnalytics({
  websiteId: 'YOUR_WEBSITE_ID',
  beforeSend: (type, payload) => {
    // 记录发送内容（用于调试）
    console.log('发送到 Umami:', type, payload)

    // 过滤敏感数据
    if (payload.url && payload.url.includes('private')) {
      return false // 取消发送
    }

    // 发送前修改数据
    return {
      ...payload,
      referrer: '' // 隐私考虑移除来源
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
