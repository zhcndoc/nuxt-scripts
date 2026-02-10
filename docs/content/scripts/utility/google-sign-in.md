---
title: Google 登录
description: 在您的 Nuxt 应用中添加支持 One Tap 和个性化按钮的 Google 登录。
links:
- label: 源代码
  icon: i-simple-icons-github
  to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-sign-in.ts
  size: xs
- label: Google 身份服务
  icon: i-simple-icons-google
  to: https://developers.google.com/identity/gsi/web/guides/overview
  size: xs
---

[Google 登录](https://developers.google.com/identity/gsi/web) 提供了一种安全且便捷的方式，让用户使用他们的 Google 账号通过 One Tap、个性化按钮及自动登录进入您的应用。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptGoogleSignIn`，可让您轻松地在 Nuxt 应用中集成 Google 登录，并实现最佳性能。

## 实时演示

::google-sign-in-demo
::

## 性能优化

该脚本以 `async` 和 `defer` 属性加载，避免阻塞渲染，确保核心网页指标（Core Web Vitals）达到最佳。初始化会延迟到您显式调用 `initialize()` 时执行，赋予您完全控制 One Tap 出现时机的能力。

## Nuxt 配置设置

在 Nuxt 应用中全局加载 Google 登录的最简单方法是使用 Nuxt 配置。或者您也可以直接使用 [useScriptGoogleSignIn](#usescriptgooglesignin) 组合式函数。

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleSignIn: {
        clientId: 'YOUR_GOOGLE_CLIENT_ID'
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
        googleSignIn: {
          clientId: 'YOUR_GOOGLE_CLIENT_ID'
        }
      }
    }
  }
})
```

::

### 使用环境变量配置

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleSignIn: true,
    }
  },
  runtimeConfig: {
    public: {
      scripts: {
        googleSignIn: {
          clientId: '', // NUXT_PUBLIC_SCRIPTS_GOOGLE_SIGN_IN_CLIENT_ID
        },
      },
    },
  },
})
```

## useScriptGoogleSignIn

`useScriptGoogleSignIn` 组合函数让您可以细粒度控制何时以及如何加载 Google 登录。

```ts
const { onLoaded } = useScriptGoogleSignIn({
  clientId: 'YOUR_GOOGLE_CLIENT_ID'
})

// 准备好后初始化
onLoaded(({ accounts }) => {
  accounts.id.initialize({
    client_id: 'YOUR_GOOGLE_CLIENT_ID',
    callback: handleCredentialResponse
  })
})
```

请参阅[注册脚本](/docs/guides/registry-scripts)指南，了解更多高级用法。

### GoogleSignInApi

```ts
export interface GoogleSignInApi {
  accounts: {
    id: {
      initialize: (config: IdConfiguration) => void
      prompt: (momentListener?: (notification: MomentNotification) => void) => void
      renderButton: (parent: HTMLElement, options: GsiButtonConfiguration) => void
      disableAutoSelect: () => void
      cancel: () => void
      revoke: (hint: string, callback: (response: RevocationResponse) => void) => void
    }
  }
}
```

### 配置参数结构

您可以通过以下选项配置 Google 登录脚本：

```ts
export const GoogleSignInOptions = object({
  clientId: string(),
  autoSelect: optional(boolean()),
  context: optional(union([literal('signin'), literal('signup'), literal('use')])),
  useFedcmForPrompt: optional(boolean()),
  cancelOnTapOutside: optional(boolean()),
  uxMode: optional(union([literal('popup'), literal('redirect')])),
  loginUri: optional(string()),
  itpSupport: optional(boolean()),
  allowedParentOrigin: optional(union([string(), array(string())])),
  hd: optional(string()), // 限制登录至 Google Workspace 域
})
```

## 示例

### One Tap 登录

One Tap 提示提供了简洁的登录体验：

```vue
<script setup lang="ts">
const { onLoaded } = useScriptGoogleSignIn()

function handleCredentialResponse(response: CredentialResponse) {
  // 将凭证发送到后端进行验证
  await $fetch('/api/auth/google', {
    method: 'POST',
    body: { credential: response.credential }
  })
}

onMounted(() => {
  onLoaded(({ accounts }) => {
    accounts.id.initialize({
      client_id: 'YOUR_CLIENT_ID',
      callback: handleCredentialResponse,
      context: 'signin',
      ux_mode: 'popup',
      use_fedcm_for_prompt: true // 使用隐私沙箱 FedCM API
    })
    
    // 显示 One Tap
    accounts.id.prompt()
  })
})
</script>
```

### 个性化按钮

渲染 Google 的个性化 “使用 Google 登录” 按钮：

```vue
<script setup lang="ts">
const { onLoaded } = useScriptGoogleSignIn()

function handleCredentialResponse(response: CredentialResponse) {
  console.log('登录成功！', response.credential)
}

onMounted(() => {
  onLoaded(({ accounts }) => {
    accounts.id.initialize({
      client_id: 'YOUR_CLIENT_ID',
      callback: handleCredentialResponse
    })
    
    const buttonDiv = document.getElementById('g-signin-button')
    if (buttonDiv) {
      accounts.id.renderButton(buttonDiv, {
        type: 'standard',
        theme: 'outline',
        size: 'large',
        text: 'signin_with',
        shape: 'rectangular',
        logo_alignment: 'left'
      })
    }
  })
})
</script>

<template>
  <div id="g-signin-button" />
</template>
```

## 状态通知

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

```bash [.env]
NUXT_PUBLIC_SCRIPTS_GOOGLE_SIGN_IN_CLIENT_ID=your-client-id.apps.googleusercontent.com
```

## 指南

::note
有关如何获取 Google 客户端 ID 及配置 OAuth 同意屏幕的详细信息，请参阅官方 [Google 身份服务文档](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid)。
::