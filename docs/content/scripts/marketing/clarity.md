---
title: Clarity
description: 在你的 Nuxt 应用中使用 Clarity。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/clarity.ts
  size: xs
---

[Clarity](https://clarity.microsoft.com/) 是微软提供的一款屏幕录制和热图工具，帮助你了解用户如何与网站交互。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptClarity`，让你可以轻松地在 Nuxt 应用中集成 Clarity。

在 Nuxt 应用中全局加载 Clarity 的最简单方式是通过 Nuxt 配置。或者你也可以直接使用 [useScriptClarity](#useScriptClarity) 组合式函数。

## 全局加载

如果你不打算发送自定义事件，可以使用 [环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 来在开发环境中禁用脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      clarity: {
        id: 'YOUR_ID'
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
        clarity: {
          id: 'YOUR_ID',
        }
      }
    }
  }
})
```


```ts [环境变量]
export default defineNuxtConfig({
  scripts: {
    registry: {
      clarity: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        clarity: {
          // .env 文件中
          // NUXT_PUBLIC_SCRIPTS_CLARITY_ID=<your-id>
          id: '',
        },
      },
    },
  },
})
```

::

## useScriptClarity

`useScriptClarity` 组合式函数允许你精细控制 Clarity 在你的网站上何时及如何加载。

```ts
const { proxy } = useScriptClarity({
  id: 'YOUR_ID'
})
// 示例
proxy.clarity("identify", "custom-id", "custom-session-id", "custom-page-id", "friendly-name")	
```

请参阅 [注册脚本](/docs/guides/registry-scripts) 指南以了解更多高级用法。

### ClarityApi

```ts
type ClarityFunctions = ((fn: 'start', options: { content: boolean, cookies: string[], dob: number, expire: number, projectId: string, upload: string }) => void)
  & ((fn: 'identify', id: string, session?: string, page?: string, userHint?: string) => Promise<{
  id: string
  session: string
  page: string
  userHint: string
}>)
  & ((fn: 'consent') => void)
  & ((fn: 'set', key: any, value: any) => void)
  & ((fn: 'event', value: any) => void)
  & ((fn: 'upgrade', upgradeReason: any) => void)
  & ((fn: string, ...args: any[]) => void)

export interface ClarityApi {
  clarity: ClarityFunctions & {
    q: any[]
    v: string
  }
}

```

### 配置 Schema

首次设置脚本时必须提供选项。

```ts
export const ClarityOptions = object({
  /**
   * Clarity 令牌。
   */
  id: pipe(string(), minLength(10)),
})
```

## 示例

仅在生产环境使用 Clarity，同时使用 `clarity` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptClarity()

// 在开发环境和 SSR 中无操作
// 在生产环境和客户端正常工作
function sendConversion() {
  proxy.clarity('event', 'conversion')
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