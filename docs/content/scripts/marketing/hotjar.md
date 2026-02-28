---
title: Hotjar
description: 在你的 Nuxt 应用中使用 Hotjar。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/hotjar.ts
  size: xs
---

[Hotjar](https://www.hotjar.com/) 是一款屏幕录制和热力图工具，帮助你了解用户如何与网站交互。

在你的 Nuxt 应用中，全局加载 Hotjar 最简单的方式是使用 Nuxt 配置。或者你也可以直接使用 [useScriptHotjar](#useScriptHotjar) 组合式函数。

## 全局加载

如果你不打算发送自定义事件，可以使用 [环境覆盖](https://nuxt.com/docs/getting-started/configuration#environment-overrides) 来在开发环境禁用该脚本。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      hotjar: {
        id: 123456, // 你的 ID
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
        hotjar: {
          id: 123456, // 你的 ID
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
      hotjar: true,
    }
  },
  // 你需要提供运行时配置以访问环境变量
  runtimeConfig: {
    public: {
      scripts: {
        hotjar: {
          id: 123456, // NUXT_PUBLIC_SCRIPTS_HOTJAR_ID
        },
      },
    },
  },
})
```

::

## useScriptHotjar

`useScriptHotjar` 组合式函数让你可以更细粒度地控制 Hotjar 在你的网站上何时以及如何加载。

```ts
const { proxy } = useScriptHotjar({
  id: 123546,
})
// 示例
proxy.hj('identify', 123456, {
  name: 'John Doe',
  email: 'john@doe.com'
})
```

请参阅 [注册脚本指南](/docs/guides/registry-scripts) 了解更多高级用法。

### HotjarApi

```ts
export interface HotjarApi {
  hj: ((event: 'identify', userId: string, attributes?: Record<string, any>) => void)
  & ((event: 'stateChange', path: string) => void)
  & ((event: 'event', eventName: string) => void)
  & ((event: string, arg?: string) => void)
  & ((...params: any[]) => void) & {
    q: any[]
  }
}
```

### 配置 Schema

首次设置脚本时必须提供选项。

```ts
export const HotjarOptions = object({
  id: number(),
  sv: optional(number()),
})
```

## 示例

该脚本支持[第一方模式](/docs/guides/first-party)，通过你的域名路由所有流量，以提升隐私保护并绕过广告拦截器。

当通过 `scripts.firstParty: true` 全局启用时，该脚本将：

- 从你的域名加载而非 `static.hotjar.com`
- 通过你的服务器路由配置和数据请求（`vars.hotjar.com`、`in.hotjar.com`）
- 匿名化 IP 地址、语言和硬件指纹（canvas、webgl、浏览器版本）
- 保留 User-Agent、屏幕分辨率和时区，以确保热力图和设备过滤的准确性

::callout{type="info"}
Hotjar 使用 WebSocket 连接进行会话录制数据。代理负责初始设置，但 WebSocket 连接直接连接到 Hotjar 服务器。
::

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    firstParty: true,
    registry: {
      hotjar: { id: 123456 }
    }
  }
})
```

若要针对该脚本选择退出：

```ts
useScriptHotjar({
  id: 123456,
  scriptOptions: {
    firstParty: false // 直接从 Hotjar 加载
  }
})
```

## 示例

仅在生产环境使用 Hotjar，并通过 `hj` 发送转化事件。

::code-group

```vue [ConversionButton.vue]
<script setup lang="ts">
const { proxy } = useScriptHotjar()

// 在开发环境和 SSR 中无操作
// 在生产环境客户端生效
function sendConversion() {
  proxy.hj('event', 'conversion')
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
