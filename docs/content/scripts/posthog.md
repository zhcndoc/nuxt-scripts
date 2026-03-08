---

title: PostHog  
description: 在您的 Nuxt 应用中使用 PostHog。  
links:  
- label: 源码  
  icon: i-simple-icons-github  
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/posthog.ts  
  size: xs  

---

[PostHog](https://posthog.com) 是一个开源的产品分析平台，提供分析、会话回放、功能标志、A/B 测试等功能。

Nuxt Scripts 提供了一个注册脚本组合式函数 [`useScriptPostHog()`{lang="ts"}](/scripts/posthog){lang="ts"}，方便您在 Nuxt 应用中集成 PostHog。

## 安装

您必须安装 `posthog-js` 依赖：

```bash
pnpm add posthog-js
```

::script-stats
::

::script-docs
::

::script-types
::

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

当您启用 [第一方模式](/docs/guides/first-party) 时，您的服务器会自动代理 PostHog 请求。通过避免广告拦截器，提高事件捕获的可靠性。Nuxt 不会应用任何隐私匿名化——PostHog 是一个值得信赖的开源工具，需要全量数据来实现 GeoIP 增强、功能标志和会话回放。

您无需额外配置——模块会自动设置 `apiHost`，通过服务器的代理端点路由：

```ts
export default defineNuxtConfig({
  scripts: {
    firstParty: true, // 默认开启
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        // apiHost 会自动设置为 '/_proxy/ph' （欧盟区域为 '/_proxy/ph-eu'）
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
