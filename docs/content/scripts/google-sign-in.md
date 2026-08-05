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

[Google 登录](https://developers.google.com/identity/gsi/web)支持 One Tap、个性化按钮，以及使用 Google 帐号自动登录。

[`useScriptGoogleSignIn()`{lang="ts"}](/scripts/google-sign-in){lang="ts"}加载 Google 身份服务，并添加用于初始化、按钮和 One Tap 提示的辅助功能。

::script-stats
::

::script-docs{:sections='["setup", "composable"]'}
::

## 在线演示

::google-sign-in-demo
::

## 可组合 API

`useScriptGoogleSignIn()`{lang="ts"} 返回标准脚本上下文（`status`、`proxy`、`onLoaded` 等）以及三个封装最常见流程的辅助函数。每次调用辅助函数时，都会合并传递给组合式函数的 schema 选项，因此无需重复设置 `clientId`、`loginUri`、`uxMode` 及相关选项。

```ts
const { initialize, renderButton, prompt, status, onLoaded, proxy } = useScriptGoogleSignIn({
  clientId: 'YOUR_CLIENT_ID',
  context: 'signin',
})
```

### `initialize(config?)`{lang="ts"}

调用 `google.accounts.id.initialize()`{lang="ts"}，并将 schema 选项与 `config` 合并。该辅助函数只会转发第一次调用，这遵循 Google 关于[每个页面只初始化一次](https://developers.google.com/identity/gsi/web/reference/js-reference#method_google.accounts.id.initialize)的指导原则，也能避免组件重新挂载时重置活动配置。

```ts
initialize({
  callback: (response) => {
    // 在服务器端验证 response.credential
  }
})
```

### `renderButton(parent, config?)`{lang="ts"}

渲染个性化按钮，并且可以安全地在语言环境变化或页面导航时重新渲染。如果 Google Identity Services 尚未初始化，该辅助函数会尝试根据已配置的选项进行初始化。弹出窗口模式需要回调函数，可以通过组合式函数或 `initialize({ callback })`{lang="ts"} 提供；如果没有回调函数，`renderButton()`{lang="ts"} 将直接返回而不进行渲染。重定向模式可以在没有回调函数的情况下完成初始化。

```vue
<script setup lang="ts">
const { initialize, renderButton } = useScriptGoogleSignIn()
const buttonRef = useTemplateRef<HTMLDivElement>('buttonRef')

initialize({
  callback: response => console.log('Credential received', response),
})

watch(buttonRef, (el) => {
  if (el)
    renderButton(el, { text: 'continue_with' })
}, { immediate: true })
</script>

<template>
  <div ref="buttonRef" />
</template>
```

### `prompt(listener?)`{lang="ts"}

显示 One Tap 提示。在弹出窗口模式下，请先调用 `initialize({ callback })`{lang="ts"}（或将回调函数传递给组合式函数）；否则初始化会被延迟，`prompt()`{lang="ts"} 将直接返回而不显示 One Tap。

```ts
prompt()
```

### 切换语言环境

按钮的[语言环境是 `renderButton` 选项](https://developers.google.com/identity/gsi/web/reference/js-reference#locale)，而不是 `initialize` 选项。要更改语言，请清空容器并重新渲染：

```ts
watch([locale, buttonRef], ([newLocale, el]) => {
  if (!el)
    return
  el.innerHTML = ''
  renderButton(el, { locale: newLocale })
}, { immediate: true })
```

### 重定向 UX 模式

使用 `uxMode: 'redirect'` 时，Google 会将凭证以 `application/x-www-form-urlencoded` 格式 **POST** 到您的 `loginUri` 服务器端点（字段包括：`credential`、`g_csrf_token`、`select_by` 等）。重定向后，凭证**不会**出现在 URL 片段中；它会随 POST 请求正文传输，由您的服务器处理后再重定向浏览器。在接受凭证之前，请验证[双重提交 CSRF 令牌](https://developers.google.com/identity/gsi/web/guides/verify-google-id-token#verify_the_cross-site_request_forgery_csrf_token)。

如果您需要在客户端获取凭证（例如使用独立 API 的 SPA），请改用带有 `callback` 的 `uxMode: 'popup'`。

```ts
const { initialize, renderButton } = useScriptGoogleSignIn({
  uxMode: 'redirect',
  loginUri: 'https://your-server.com/auth/google',
})

initialize() // 重定向模式下不需要 callback
```

## Moment 通知

使用 FedCM 时，Google 会移除显示时刻通知和详细的跳过原因。Google 还警告称，提示回调可能不会接收到每个时刻通知，因此不要让应用程序流程依赖于它。如果你检查剩余的跳过和关闭时刻，请避免使用已移除的方法：

```ts
const { initialize, prompt } = useScriptGoogleSignIn()

initialize({
  callback: response => console.log('Credential received', response),
})
prompt((notification) => {
  if (notification.isSkippedMoment()) {
    console.log('One Tap skipped')
  }

  if (notification.isDismissedMoment()) {
    console.log('Dismissed:', notification.getDismissedReason())
  }
})
```

请参阅 Google 的 [FedCM 迁移指南](https://developers.google.com/identity/gsi/web/guides/fedcm-migration#remove_use_of_isdisplaymoment_isdisplayed_isnotdisplayed_and_getnotdisplayedreason_methods)，了解已移除的方法和通知限制。

## 服务端验证

始终在[您的服务器上验证凭据令牌](https://developers.google.com/identity/gsi/web/guides/verify-google-id-token)。Google 的 [Node.js 身份验证库](https://github.com/googleapis/google-auth-library-nodejs)会检查签名以及 `aud`、`iss` 和 `exp` 声明：

```bash
pnpm add google-auth-library
```

```ts [server/api/auth/google.post.ts]
import { OAuth2Client } from 'google-auth-library'

const client = new OAuth2Client()

export default defineEventHandler(async (event) => {
  const { credential } = await readBody(event)

  const ticket = await client.verifyIdToken({
    idToken: credential,
    audience: 'YOUR_CLIENT_ID',
  })
  const payload = ticket.getPayload()
  if (!payload) {
    throw createError({ statusCode: 401, message: 'Invalid token' })
  }

  const user = {
    email: payload.email,
    name: payload.name,
    picture: payload.picture,
    sub: payload.sub,
  }

  return { user }
})
```

## 跨源打开者策略

非 FedCM 的 `popup` 流程可能需要兼容的 `Cross-Origin-Opener-Policy`，以便弹出窗口能够与您的页面通信。

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

Google 的 [COOP 设置指南](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid#cross_origin_opener_policy)会在您停用 FedCM 时应用此标头。浏览器渲染的 FedCM 弹出窗口和重定向模式不需要此设置。

## FedCM API 支持

Google 现在将 `use_fedcm_for_prompt` 标记为[已弃用且会被忽略](https://developers.google.com/identity/gsi/web/reference/js-reference#use_fedcm_for_prompt)。注册表中的 `useFedcmForPrompt` 选项仍映射到该字段，因此更改它不会产生任何效果。按钮 FedCM 使用 `use_fedcm_for_button`，而该注册表目前尚未映射到此字段。

### 跨源 iframe

对于[受支持的同站点跨源 iframe 集成](https://developers.google.com/identity/gsi/web/guides/fedcm-migration#add_allowidentity-credentials-get_attribute_to_parent_frame_if_your_web_app_calls_one_tap_or_button_api_from_cross-origin_iframes)，请将 `allow` 属性添加到每个父级 iframe：

```html
<iframe src="https://your-app.com/login" allow="identity-credentials-get"></iframe>
```

::warning
启用 FedCM 后，不支持使用 `prompt_parent_id` 自定义 One Tap 提示框的位置。Google 不支持在跨站点 iframe 中使用 One Tap。
::

## 撤销使用 Google 登录的授权

在[撤销使用 Google 登录的授权](https://developers.google.com/identity/gsi/web/guides/revoke)时，使用电子邮件地址或 Google 用户 ID 作为提示信息：

```ts
const { onLoaded } = useScriptGoogleSignIn()

function revokeAccess(hint: string) {
  onLoaded(({ accounts }) => {
    accounts.id.revoke(hint, (response) => {
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

如果启用了自动登录，请在用户登出时调用 [`disableAutoSelect()`{lang="ts"}](https://developers.google.com/identity/gsi/web/guides/automatic-sign-in-sign-out#sign-out)。这样可以防止同一账户立即再次登录：

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

### 托管域名限制

使用 `hd` 优化 Google Workspace 域名的登录流程：

```ts
const { initialize } = useScriptGoogleSignIn({
  hd: 'your-company.com',
})

initialize({ callback: handleCredentialResponse })
```

客户端的 `hd` 选项不是授权检查。在限制对该域名的访问之前，请在服务器上[验证 ID 令牌的 `hd` 声明](https://developers.google.com/identity/gsi/web/guides/verify-google-id-token)。

## 本地开发设置

要在本地测试 Google 登录：

1. 前往 [Google Cloud Console → 凭据](https://console.cloud.google.com/apis/credentials)
2. 创建或选择一个 OAuth 2.0 客户端 ID（Web 应用类型）
3. 在 **已获授权的 JavaScript 来源** 下，添加：
   - `http://localhost:3000`（或你的确切开发服务器来源）
4. 保存并复制你的客户端 ID

::note
添加确切的开发来源，包括其协议和端口。使用弹出窗口模式时不需要重定向 URI。
::

然后配置环境变量：

```bash
NUXT_PUBLIC_SCRIPTS_GOOGLE_SIGN_IN_CLIENT_ID=your-client-id.apps.googleusercontent.com
```

## 指南

::note
请参阅 [Google 的设置指南](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid) 以创建客户端 ID 并配置 OAuth 同意屏幕。
::

::script-types
::

## 示例

### 一键登录

初始化 Google Identity Services，然后打开一键提示：

```vue
<script setup lang="ts">
const { initialize, prompt } = useScriptGoogleSignIn({
  context: 'signin',
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

渲染个性化的“使用 Google 登录”按钮：

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
