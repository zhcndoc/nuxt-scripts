---
title: Google 登录
description: 为您的 Nuxt 应用添加 Google 登录，支持 One Tap 和个性化按钮。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-sign-in.ts
  size: xs
- label: Google 身份服务
  icon: i-simple-icons-google
  to: https://developers.google.com/identity/gsi/web/guides/overview
  size: xs
---

[Google 登录](https://developers.google.com/identity/gsi/web) 提供了一种安全且便捷的方式，让用户使用他们的 Google 账号通过 One Tap、个性化按钮及自动登录进入您的应用。

Nuxt Scripts 提供了一个注册表脚本组合式函数 [`useScriptGoogleSignIn()`{lang="ts"}](/scripts/google-sign-in){lang="ts"}，可轻松集成 Google 登录到您的 Nuxt 应用中，并具备最佳性能。

::script-stats
::

::script-docs{:sections='["setup", "composable"]'}
::

## 实时演示

::google-sign-in-demo
::

## Composable API

`useScriptGoogleSignIn()`{lang="ts"} 返回标准脚本上下文（`status`、`proxy`、`onLoaded` 等）以及三个帮助函数，用于封装最常见的流程。每次调用帮助函数时，都会与传递给 composable 的 schema 选项合并，因此您无需重复传入 `clientId`、`loginUri`、`uxMode` 等参数。

```ts
const { initialize, renderButton, prompt, status, onLoaded, proxy } = useScriptGoogleSignIn({
  clientId: 'YOUR_CLIENT_ID',
  context: 'signin',
  useFedcmForPrompt: true,
})
```

### `initialize(config?)`{lang="ts"}

使用与 `config` 合并后的 schema 选项调用 `google.accounts.id.initialize()`{lang="ts"}。内部已做防护，因此多次调用（例如跨导航和组件重新挂载）是安全的；如果 `initialize()`{lang="ts"} 在页面渲染按钮后再次运行，Google 的 API 会记录错误，因此该帮助函数只会转发第一次调用。

```ts
initialize({
  callback: (response) => {
    // 在服务器端验证 response.credential
  }
})
```

### `renderButton(parent, config?)`{lang="ts"}

渲染个性化按钮。如有需要会自动初始化，并且在语言切换或导航时可安全重新渲染。

```vue
<script setup lang="ts">
const { initialize, renderButton } = useScriptGoogleSignIn()
const buttonRef = useTemplateRef<HTMLDivElement>('buttonRef')

initialize({ callback: handleCredential })

watch(buttonRef, (el) => {
  if (el)
    renderButton(el, { text: 'continue_with', use_fedcm: true })
}, { immediate: true })
</script>

<template>
  <div ref="buttonRef" />
</template>
```

### `prompt(listener?)`{lang="ts"}

显示 One Tap 提示。如有需要会自动初始化。

```ts
prompt((notification) => {
  if (notification.isNotDisplayed())
    console.log('未显示:', notification.getNotDisplayedReason())
})
```

### 切换语言环境

按钮的语言环境是 `renderButton` 的选项，而不是 `initialize` 的选项。要更改语言，请清空容器并重新渲染：

```ts
watch([locale, buttonRef], ([newLocale, el]) => {
  if (!el)
    return
  el.innerHTML = ''
  renderButton(el, { locale: newLocale, use_fedcm: true })
}, { immediate: true })
```

::note
即使设置了此选项，Google 仍可能回退到用户的 Google 账户语言。这是 Google 端的行为，无法配置。
::

### 重定向 UX 模式

在 `uxMode: 'redirect'` 下，Google 会将凭证通过 **POST** 以 `application/x-www-form-urlencoded` 的形式发送到您的 `loginUri` 服务器端点（字段包括：`credential`、`g_csrf_token`、`select_by` 等）。重定向后，凭证不会作为 URL fragment 出现；它会随 POST 请求体传输，由您的服务器在将浏览器重定向之前进行处理。

如果您需要在客户端获取凭证（例如使用独立 API 的 SPA），请改用带有 `callback` 的 `uxMode: 'popup'`。

```ts
const { initialize, renderButton } = useScriptGoogleSignIn({
  uxMode: 'redirect',
  loginUri: 'https://your-server.com/auth/google',
})

initialize() // 重定向模式下不需要 callback
```

## Moment 通知

跟踪 One Tap 的显示状态：

```ts
const { onLoaded } = useScriptGoogleSignIn()

onLoaded(({ accounts }) => {
  accounts.id.prompt((notification) => {
    if (notification.isDisplayMoment()) {
      if (notification.isDisplayed()) {
        console.log('One Tap 已显示')
      } else {
        console.log('未显示:', notification.getNotDisplayedReason())
      }
    }

    if (notification.isSkippedMoment()) {
      console.log('已跳过:', notification.getSkippedReason())
    }

    if (notification.isDismissedMoment()) {
      console.log('已关闭:', notification.getDismissedReason())
    }
  })
})
```

## 服务器端验证

始终在服务器端验证凭证令牌：

```ts [server/api/auth/google.post.ts]
export default defineEventHandler(async (event) => {
  const { credential } = await readBody(event)
  
  // 向 Google 验证令牌
  const response = await $fetch(`https://oauth2.googleapis.com/tokeninfo`, {
    params: { id_token: credential }
  })
  
  // 验证客户端 ID 是否匹配
  if (response.aud !== 'YOUR_CLIENT_ID') {
    throw createError({ statusCode: 401, message: '无效的令牌' })
  }
  
  // 使用用户信息创建会话
  const user = {
    email: response.email,
    name: response.name,
    picture: response.picture,
    sub: response.sub
  }

  return { user }
})
```

## Cross-Origin-Opener-Policy

在 `popup` UX 模式（默认）下，Google 会打开一个弹窗，并通过 `window.postMessage` 将凭证返回到您的页面。如果您的页面发送了严格的 `Cross-Origin-Opener-Policy` 标头，浏览器会阻止该消息，登录看起来就像没有反应：弹窗会关闭，但不会触发回调。

如果您设置了 COOP，请使用 `same-origin-allow-popups`：

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  routeRules: {
    '/login/**': {
      headers: { 'Cross-Origin-Opener-Policy': 'same-origin-allow-popups' },
    },
  },
})
```

未显式设置 COOP 标头的页面默认可正常工作。重定向模式（`uxMode: 'redirect'`）和仅 FedCM 流程不受影响。

## FedCM API 支持

启用隐私沙箱 [FedCM API](https://developers.google.com/identity/gsi/web/guides/fedcm-migration) 支持以增强隐私保护。**2025 年 8 月起将强制采用 FedCM。**

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleSignIn: {
        clientId: 'YOUR_CLIENT_ID',
        useFedcmForPrompt: true
      }
    }
  }
})
```

### 跨域 iframe

在使用 FedCM 的 One Tap 或登录按钮于跨域 iframe 中时，应为所有父 iframe 添加 `allow` 属性：

```html
<iframe src="https://your-app.com/login" allow="identity-credentials-get"></iframe>
```

::warning
启用 FedCM 后，无法通过 `prompt_parent_id` 定制 One Tap 提示的位置。
::

## 撤销访问权限

允许用户撤销对其 Google 账户的访问权限：

```ts
const { onLoaded } = useScriptGoogleSignIn()

function revokeAccess(userId: string) {
  onLoaded(({ accounts }) => {
    accounts.id.revoke(userId, (response) => {
      if (response.successful) {
        console.log('访问权限已撤销')
      } else {
        console.error('撤销失败:', response.error)
      }
    })
  })
}
```

## 最佳实践

### 登出处理

用户登出时务必调用 `disableAutoSelect()`，防止自动重新认证：

```ts
function signOut() {
  // 清除应用会话
  user.value = null

  // 阻止 One Tap 自动选择此账户
  onLoaded(({ accounts }) => {
    accounts.id.disableAutoSelect()
  })
}
```

### 限制托管域

限制登录仅允许来自特定 Google Workspace 域的用户：

```ts
accounts.id.initialize({
  client_id: 'YOUR_CLIENT_ID',
  callback: handleCredentialResponse,
  hd: 'your-company.com' // 仅允许该域用户登录
})
```

## 本地开发配置

要在本地测试 Google 登录：

1. 打开 [Google Cloud 控制台 → 凭据](https://console.cloud.google.com/apis/credentials)
2. 创建或选择一个 OAuth 2.0 客户端 ID（Web 应用类型）
3. 在 **授权的 JavaScript 来源** 中添加：
   - `http://localhost`
   - `http://localhost:3000`（或您的开发服务器端口）
4. 保存并复制客户端 ID

::note
Google 要求本地开发使用 `http://localhost`（而非 `127.0.0.1`）。使用弹出模式时无需重定向 URI。
::

然后配置环境变量：

```bash
NUXT_PUBLIC_SCRIPTS_GOOGLE_SIGN_IN_CLIENT_ID=your-client-id.apps.googleusercontent.com
```

## 指南

::note
有关如何获取 Google 客户端 ID 及配置 OAuth 同意屏幕的详细信息，请参阅官方 [Google 身份服务文档](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid)。
::

::script-types
::

## 示例

### One Tap 登录

One Tap 提示提供了一种简化的登录体验：

```vue
<script setup lang="ts">
const { initialize, prompt } = useScriptGoogleSignIn({
  context: 'signin',
  useFedcmForPrompt: true,
})

async function handleCredentialResponse(response: CredentialResponse) {
  await $fetch('/api/auth/google', {
    method: 'POST',
    body: { credential: response.credential }
  })
}

initialize({ callback: handleCredentialResponse })
onMounted(() => prompt())
</script>
```

### 个性化按钮

渲染 Google 的个性化“使用 Google 登录”按钮：

```vue
<script setup lang="ts">
const { initialize, renderButton } = useScriptGoogleSignIn()
const buttonRef = useTemplateRef<HTMLDivElement>('buttonRef')

function handleCredentialResponse(response: CredentialResponse) {
  console.log('已登录!', response.credential)
}

initialize({ callback: handleCredentialResponse })

watch(buttonRef, (el) => {
  if (el) {
    renderButton(el, {
      type: 'standard',
      theme: 'outline',
      size: 'large',
      text: 'signin_with',
      shape: 'rectangular',
      logo_alignment: 'left',
    })
  }
}, { immediate: true })
</script>

<template>
  <div ref="buttonRef" />
</template>
```
