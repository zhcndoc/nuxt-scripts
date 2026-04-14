---
title: Gravatar
description: 在你的 Nuxt 应用中使用 Gravatar。
links:
- label: 源码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/gravatar.ts
  size: xs
- label: "<ScriptGravatar>"
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/components/ScriptGravatar.vue
  size: xs
---

[Gravatar](https://gravatar.com) 提供与邮箱地址关联的全球通用头像。Nuxt Scripts 提供了一种保护隐私的集成方案，通过你自己的服务器代理头像请求，防止 Gravatar 追踪你的用户。

::script-stats  
::

::script-docs  
::

::callout{type="info"}
当配置了 `NUXT_SCRIPTS_PROXY_SECRET` 时，该脚本的代理端点会使用 [HMAC URL 签名](/docs/guides/first-party#proxy-endpoint-security)。请参阅 [安全指南](/docs/guides/first-party#proxy-endpoint-security) 获取设置说明。
::

## [`<ScriptGravatar>`{lang="html"}](/scripts/gravatar){lang="html"}

[`<ScriptGravatar>`{lang="html"}](/scripts/gravatar){lang="html"} 组件会为给定的邮箱地址渲染 Gravatar 头像。所有请求都会通过你的服务器代理发送 — Gravatar 永远不会看到你的用户的 IP 地址或请求头。

### 演示

::code-group

:gravatar-demo{label="输出"}

```vue [输入]
<template>
  <ScriptGravatar
    email="info@gravatar.com"
    :size="80"
    class="rounded-full"
  />
</template>
```

::

### 组件 API

请查看 [Facade Component API](/docs/guides/facade-components#facade-components-api) 了解完整的属性、事件和插槽。

### 属性

| 属性 | 类型 | 默认值 | 说明 |
|------|------|---------|-------------|
| `email` | `string` | - | 邮箱地址，发送到你的服务器代理进行哈希处理，不会发送给 Gravatar |
| `hash` | `string` | - | 预先计算的邮箱 SHA256 哈希值（`email` 的替代选项） |
| `size` | `number` | `80` | 头像尺寸（像素） |
| `default` | `string` | `'mp'` | 当没有 Gravatar 存在时的默认头像样式 |
| `rating` | `string` | `'g'` | 内容评级过滤器 |
| `hovercards` | `boolean` | `false` | 悬停时启用悬停卡片 |

## [`useScriptGravatar()`{lang="ts"}](/scripts/gravatar){lang="ts"}

[`useScriptGravatar()`{lang="ts"}](/scripts/gravatar){lang="ts"} 组合式函数让你能够以编程方式与 Gravatar API 交互。

```ts
export function useScriptGravatar<T extends GravatarApi>(_options?: GravatarInput) {}
```

请遵循 [Registry Scripts](/docs/guides/registry-scripts) 指南以了解更多高级用法。

::script-types
::

## 示例

使用组合式函数直接获取头像 URL。

```vue
<script setup lang="ts">
const { onLoaded } = useScriptGravatar()

const avatarUrl = ref('')

onLoaded((api) => {
  avatarUrl.value = api.getAvatarUrlFromEmail('user@example.com', { size: 120 })
})
</script>

<template>
  <img v-if="avatarUrl" :src="avatarUrl" alt="User avatar">
</template>
```
