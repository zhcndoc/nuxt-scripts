---
title: Segment
description: 在你的 Nuxt 应用中使用 Segment。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/segment.ts
  size: xs
---

[Segment](https://segment.com/) 让你能够收集、清理和控制客户数据。Segment 帮助你了解客户并个性化他们的体验。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptSegment`，可以轻松地在你的 Nuxt 应用中集成 Segment。

### Nuxt 配置设置

在 Nuxt 应用中全局加载 Segment 最简便的方法是使用 Nuxt 配置。或者你也可以直接使用 [useScriptSegment](#useScriptSegment) 组合式函数。

如果你不打算发送自定义事件，可以使用[环境变量覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides)来在开发环境中禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      segment: {
        writeKey: 'YOUR_WRITE_KEY'
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
        segment: {
          writeKey: 'YOUR_WRITE_KEY'
        }
      }
    }
  }
})
```

::

#### 使用环境变量

如果你更倾向于使用环境变量配置你的 ID。

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      segment: true,
    }
  },
  // 你需要提供运行时配置来访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        segment: {
          writeKey: '' // NUXT_PUBLIC_SCRIPTS_SEGMENT_WRITE_KEY
        }
      },
    },
  },
})
```

```text [.env]
NUXT_PUBLIC_SCRIPTS_SEGMENT_WRITE_KEY=<YOUR_WRITE_KEY>
```

## useScriptSegment

`useScriptSegment` 组合式函数让你可以精细控制 Segment 在你的网站上何时以及如何加载。

```ts
const { proxy } = useScriptSegment({
  id: 'YOUR_ID'
})
// 示例
proxy.track('event', {
  foo: 'bar'
})
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南，了解更多高级用法。

### SegmentApi

```ts
interface SegmentApi {
  track: (event: string, properties?: Record<string, any>) => void
  page: (name?: string, properties?: Record<string, any>) => void
  identify: (userId: string, traits?: Record<string, any>, options?: Record<string, any>) => void
  group: (groupId: string, traits?: Record<string, any>, options?: Record<string, any>) => void
  alias: (userId: string, previousId: string, options?: Record<string, any>) => void
  reset: () => void
}
```

### 配置模式

在首次设置脚本时，必须提供选项。

```ts
export const SegmentOptions = object({
  writeKey: string(),
  analyticsKey: optional(string()),
})
```

## 示例

仅在生产环境中使用 Segment，同时使用 `analytics` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptSegment()

// 在开发环境和 SSR 时无操作
// 仅在生产环境客户端生效
function sendConversion() {
  proxy.track('conversion', {
    value: 1,
    currency: 'USD'
  })
}
</script>

<template>
  <div>
    <button @click="sendConversion">
      发送转化事件
    </button>
  </div>
</template>
```

::