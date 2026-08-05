---
title: Gravatar
description: 从电子邮件地址或哈希值渲染经过服务器哈希处理并代理的 Gravatar 图片。
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

[Gravatar](https://gravatar.com) 提供与电子邮件地址关联的全球通用头像。其[头像 API](https://docs.gravatar.com/sdk/images/) 使用标准化电子邮件地址的 SHA-256 哈希值。Nuxt Scripts 会在服务器上创建该哈希值，并代理图片请求。

::script-stats  
::

::script-docs  
::

## [`<ScriptGravatar>`{lang="html"}](/scripts/gravatar){lang="html"}

[`<ScriptGravatar>`{lang="html"}](/scripts/gravatar){lang="html"} 组件会为指定的电子邮件地址渲染 Gravatar 头像。头像图片请求会通过您的服务器代理，因此 Gravatar 不会从该请求中获取用户的 IP 地址。

传入 `email` 会在服务器对地址进行哈希处理之前，将原始地址放入同源图片 URL 中。该 URL 可能会出现在浏览器、CDN 和服务器访问日志中。当地址不应进入这些日志时，请传入预先计算的 `hash`。

::callout{type="warning"}
即使 `hovercards` 为 `false`，该集成仍会直接加载 Gravatar 的 `gprofiles.js` 以支持悬浮信息卡。因此，浏览器仍会连接到 Gravatar。
::

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

### Props

| 属性 | 类型 | 默认值 | 说明 |
|------|------|---------|-------------|
| `email` | `string` | - | 电子邮件地址，会发送到您的服务器代理进行哈希处理，不会发送给 Gravatar |
| `hash` | `string` | - | 预先计算的电子邮件 SHA-256 哈希值（`email` 的替代方案） |
| `size` | `number` | `80` | 头像的像素尺寸 |
| `default` | `string` | `'mp'` | 不存在 Gravatar 时使用的默认头像样式 |
| `rating` | `string` | `'g'` | 内容分级过滤器 |
| `hovercards` | `boolean` | `false` | 启用悬浮信息卡 |

## [`useScriptGravatar()`{lang="ts"}](/scripts/gravatar){lang="ts"}

[`useScriptGravatar()`{lang="ts"}](/scripts/gravatar){lang="ts"} 组合式函数可根据电子邮件地址或哈希值构建代理头像 URL。

```ts
export function useScriptGravatar<T extends GravatarApi>(_options?: GravatarInput) {}
```

有关触发和加载选项，请参阅[注册表脚本](/docs/guides/registry-scripts)。

::script-types
::

## 示例

在 Gravatar 脚本加载后构建头像 URL：

```vue
<script setup lang="ts">
const { onLoaded } = useScriptGravatar()

const avatarUrl = ref('')

onLoaded((api) => {
  avatarUrl.value = api.getAvatarUrlFromEmail('user@example.com', { size: 120 })
})
</script>

<template>
  <img v-if="avatarUrl" :src="avatarUrl" alt="用户头像">
</template>
```
