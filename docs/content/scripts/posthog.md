---
title: PostHog
description: 加载支持第一方代理、功能标志以及选择加入或退出控制的 posthog-js。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/posthog.ts
  size: xs
---

[PostHog](https://posthog.com) 是一个开源的产品分析平台，提供会话回放、功能标志和实验功能。Nuxt Scripts 从 [npm](https://www.npmjs.com/) 加载 PostHog 官方的 [`posthog-js` 浏览器 SDK](https://posthog.com/docs/libraries/js)。

使用 [`useScriptPostHog()`{lang="ts"}](/scripts/posthog){lang="ts"} 加载 SDK 并访问 PostHog 客户端。

::script-stats
::

::script-docs
::

## 安装

您必须安装 `posthog-js` 依赖：

```bash
pnpm add posthog-js
```

## 欧盟托管

要使用 PostHog 的欧盟云：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        region: 'eu'
      }
    }
  }
})
```

## 第一方代理

当[第一方模式](/docs/guides/first-party)处于启用状态时（支持该模式的脚本会自动启用），您的服务器会自动代理 PostHog 请求。PostHog [建议使用反向代理](https://posthog.com/docs/advanced/proxy)来更可靠地捕获事件，因为广告拦截器可能会拒绝发送到已知分析域名的请求。Nuxt 会将客户端 IP 地址匿名化到子网级别；其他数据会直接通过，以确保会话回放和功能标志等功能继续正常运行。

该模块会将 `apiHost` 设置为您服务器的代理端点：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        // apiHost 自动设置为 '/_scripts/p/ph'（或为欧盟区域设置为 '/_scripts/p/ph-eu'）
      }
    }
  }
})
```

该代理会处理 API 请求和静态资源，包括会话录制 SDK。

## 自定义 API 主机

要使用自定义反向代理或自托管的 PostHog 实例，请直接设置 `apiHost`：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        apiHost: '/my-proxy'
      }
    }
  }
})
```

`apiHost` 选项接受任何 URL 或相对路径，覆盖 `region` 的默认设置和第一方代理的自动配置。要传递 PostHog SDK 的其他选项（例如 `ui_host`），请使用 `config` 透传。

## 功能标志

功能标志方法会返回值，因此您需要先等待 PostHog 加载完成：

```ts
const { onLoaded } = useScriptPostHog()

onLoaded(({ posthog }) => {
  // 检查功能标志
  if (posthog.isFeatureEnabled('new-dashboard')) {
    // 显示新仪表盘
  }

  // 获取功能标志负载
  const payload = posthog.getFeatureFlagPayload('experiment-config')
})
```

## 同意模式

PostHog 提供 [`opt_in_capturing` / `opt_out_capturing`](https://posthog.com/docs/libraries/js#opt-out-of-data-capture)。使用 `defaultConsent` 设置启动时的默认值，并在运行时调用 `consent.optIn()`{lang="ts"} / `consent.optOut()`{lang="ts"}。

### `defaultConsent`

| 值 | 行为 |
|-------|-----------|
| `'opt-in'` | 在初始化后立即调用 `posthog.opt_in_capturing()`{lang="ts"}。 |
| `'opt-out'` | 调用 `posthog.init(..., { opt_out_capturing_by_default: true })`{lang="ts"}，使 SDK 以退出状态启动。 |

::callout{icon="i-heroicons-information-circle"}
当您需要 SDK 以退出状态启动时，请使用 `defaultConsent: 'opt-out'`。运行时的 `consent.optOut()`{lang="ts"} 会在初始化后调用 `opt_out_capturing()`{lang="ts"}，这比启动时标志更弱；在初始化和退出调用之间捕获的任何事件仍会被发送。
::

### 示例

```vue
<script setup lang="ts">
const { consent } = useScriptPostHog({
  apiKey: 'YOUR_API_KEY',
  defaultConsent: 'opt-out',
})

function onAccept() {
  consent.optIn()
}
function onRevoke() {
  consent.optOut()
}
</script>
```

在 `nuxt.config` 中全局配置 PostHog：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        defaultConsent: 'opt-out',
      }
    }
  }
})
```

## 禁用会话录制

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        disableSessionRecording: true
      }
    }
  }
})
```

::script-types
::
