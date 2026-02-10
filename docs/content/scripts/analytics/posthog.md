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