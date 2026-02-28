---
title: PostHog
description: 在你的 Nuxt 应用中使用 PostHog。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/posthog.ts
  size: xs
---

[PostHog](https://posthog.com) 是一个开源的产品分析平台，提供分析、会话回放、功能标记、A/B 测试等功能。

Nuxt Scripts 提供了一个注册脚本组合函数 `useScriptPostHog`，可以轻松地在你的 Nuxt 应用中集成 PostHog。

## 安装

你需要安装 `posthog-js` 依赖：

```bash
pnpm add posthog-js
```

### Nuxt 配置设置

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY'
      }
    }
  }
})
```

```ts [仅生产环境]
export default defineNuxtConfig({
  $production: {
    scripts: {
      registry: {
        posthog: {
          apiKey: 'YOUR_API_KEY'
        }
      }
    }
  }
})
```

::

#### 使用环境变量

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      posthog: true,
    }
  },
  runtimeConfig: {
    public: {
      scripts: {
        posthog: {
          apiKey: '', // NUXT_PUBLIC_SCRIPTS_POSTHOG_API_KEY
        },
      },
    },
  },
})
```

## useScriptPostHog

```ts
const { proxy } = useScriptPostHog({
  apiKey: 'YOUR_API_KEY'
})

// 捕获事件
proxy.posthog.capture('button_clicked', {
  button_name: 'signup'
})
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南，了解更多高级用法。

### PostHogApi

```ts
import type { PostHog } from 'posthog-js'

export interface PostHogApi {
  posthog: PostHog
}
```

### 配置模式

```ts
export const PostHogOptions = object({
  apiKey: string(),
  region: optional(union([literal('us'), literal('eu')])),
  apiHost: optional(string()), // 自定义 API 主机 URL（例如用于反向代理的 '/ph'）
  autocapture: optional(boolean()),
  capturePageview: optional(boolean()),
  capturePageleave: optional(boolean()),
  disableSessionRecording: optional(boolean()),
  config: optional(record(string(), any())), // 完整的 PostHogConfig 透传
})
```

## 示例

使用 PostHog 跟踪注册事件。

::code-group

```vue [SignupForm.vue]
<script setup lang="ts">
const { proxy } = useScriptPostHog()

function onSignup(email: string) {
  proxy.posthog.identify(email, {
    email,
    signup_date: new Date().toISOString()
  })
  proxy.posthog.capture('user_signed_up')
}
</script>

<template>
  <form @submit.prevent="onSignup(email)">
    <input v-model="email" type="email" />
    <button type="submit">注册</button>
  </form>
</template>
```

::

## 欧盟托管

使用 PostHog 的欧盟云：

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

## 首方代理

当启用[首方模式](/docs/guides/first-party)时，PostHog 请求会自动通过你自己的服务器代理转发。这通过绕过广告拦截器，提高了事件捕获的可靠性。不会应用隐私匿名处理——PostHog 是一个受信任的开源工具，需完整数据以进行 GeoIP 丰富化、功能标记和会话回放。

无需额外配置——该模块会自动将 `apiHost` 设置为通过你服务器的代理端点路由：

```ts
export default defineNuxtConfig({
  scripts: {
    firstParty: true, // 默认为启用
    registry: {
      posthog: {
        apiKey: 'YOUR_API_KEY',
        // apiHost 会自动设置为 '/_proxy/ph'（或针对欧盟区域为 '/_proxy/ph-eu'）
      }
    }
  }
})
```

代理会处理 API 请求和静态资源（如会话录制 SDK），路由到正确的 PostHog 端点。

## 自定义 API 主机

如需使用自定义的反向代理或自托管的 PostHog 实例，直接设置 `apiHost`：

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

`apiHost` 选项支持任何 URL 或相对路径，会覆盖 `region` 默认设置和首方代理的自动配置。对于 `ui_host` 等额外的 PostHog SDK 选项，可通过 `config` 透传。

## 功能标记

功能标记方法会返回值，因此需要先等待 PostHog 加载完成：

```ts
const { onLoaded } = useScriptPostHog()

onLoaded(({ posthog }) => {
  // 检查功能标记
  if (posthog.isFeatureEnabled('new-dashboard')) {
    // 显示新仪表盘
  }

  // 获取标记负载
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
