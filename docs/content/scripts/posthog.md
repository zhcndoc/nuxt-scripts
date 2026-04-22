---
title: PostHog
description: 在您的 Nuxt 应用中使用 PostHog。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/posthog.ts
  size: xs
---

[PostHog](https://posthog.com) 是一个开源的产品分析平台，提供分析、会话回放、功能标志、A/B 测试等功能。

Nuxt Scripts 提供了一个注册脚本组合式函数 [`useScriptPostHog()`{lang="ts"}](/scripts/posthog){lang="ts"}，方便您在 Nuxt 应用中集成 PostHog。

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

当 [第一方模式](/docs/guides/first-party) 处于活动状态时（为支持的脚本自动启用），您的服务器会自动代理 PostHog 请求。这通过避免广告拦截器来提高事件捕获的可靠性。Nuxt 不会应用隐私匿名化；PostHog 是一个受信任的开源工具，需要完整保真度的数据来进行 GeoIP 丰富、功能标志和会话回放。

无需额外配置。该模块会自动设置 `apiHost` 以通过您服务器的代理端点进行路由：

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

代理处理 API 请求和静态资源（例如会话录制 SDK），将它们路由到正确的 PostHog 端点。

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

PostHog 提供 [`opt_in_capturing` / `opt_out_capturing`](https://posthog.com/docs/privacy/opting-out)。使用 `defaultConsent` 设置启动时默认值，并在运行时调用 `consent.optIn()`{lang="ts"} / `consent.optOut()`{lang="ts"}。

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
