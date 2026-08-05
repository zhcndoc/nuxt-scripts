---
title: Google reCAPTCHA
description: 加载基于评分的 reCAPTCHA v3 或 Enterprise，并执行受保护的操作。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/packages/script/src/runtime/registry/google-recaptcha.ts
    size: xs
---

[Google reCAPTCHA](https://cloud.google.com/security/products/recaptcha) 会对请求进行评分，以判断其是否可能涉及垃圾信息和滥用行为，而不会显示复选框。

[`useScriptGoogleRecaptcha()`{lang="ts"}](/scripts/google-recaptcha){lang="ts"} 会加载所选客户端并提供 `grecaptcha`。

::callout
此注册表集成支持基于评分的 reCAPTCHA v3 和 Enterprise 流程。要渲染 v2 复选框，请使用 `useScript()`{lang="ts"} 单独加载，并遵循 Google 的 [v2 显示指南](https://developers.google.com/recaptcha/docs/display)。
::

::script-stats  
::

::script-docs{:sections='["setup", "composable"]'}  
::

## 企业版

使用 reCAPTCHA Enterprise 时，将 `enterprise` 选项设置为 `true`：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleRecaptcha: {
        siteKey: 'YOUR_SITE_KEY',
        enterprise: true
      }
    }
  }
})
```

Enterprise 将其方法暴露在 `grecaptcha.enterprise` 下，因此应使用该对象执行操作，而不是使用 `grecaptcha.execute`：

```ts
const { onLoaded } = useScriptGoogleRecaptcha({
  siteKey: 'YOUR_SITE_KEY',
  enterprise: true,
})

onLoaded(({ grecaptcha }) => {
  grecaptcha.enterprise!.ready(async () => {
    const token = await grecaptcha.enterprise!.execute('YOUR_SITE_KEY', { action: 'submit' })
    // 将令牌发送到服务器进行评估。
  })
})
```

## 替代域名

当 `google.com` 无法访问时，将 `recaptchaNet: true` 设置为启用。Google 将 `recaptcha.net` 记录为其[面向全球访问的替代域名](https://developers.google.com/recaptcha/docs/faq#can-i-use-recaptcha-globally)：

```ts
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleRecaptcha: {
        siteKey: 'YOUR_SITE_KEY',
        recaptchaNet: true
      }
    }
  }
})
```

## 服务端验证

务必在你的服务器上验证每个 reCAPTCHA 令牌。Google 建议同时检查分数和预期操作，然后根据你自己的流量调整分数阈值，而不要将 `0.5` 视为通用标准。请参阅 [reCAPTCHA v3 验证指南](https://developers.google.com/recaptcha/docs/v3#site_verify_response)。

::code-group

```ts [server/api/verify-recaptcha.post.ts]
export default defineEventHandler(async (event) => {
  const { token } = await readBody(event)
  const secretKey = process.env.RECAPTCHA_SECRET_KEY
  if (!secretKey)
    throw createError({ statusCode: 500, message: 'Missing reCAPTCHA secret key' })

  const response = await $fetch('https://www.google.com/recaptcha/api/siteverify', {
    method: 'POST',
    body: new URLSearchParams({
      secret: secretKey,
      response: token,
    }),
  })

  if (!response.success || response.action !== 'submit' || response.score < 0.5) {
    throw createError({
      statusCode: 400,
      message: 'reCAPTCHA 验证失败',
    })
  }

  return { success: true, score: response.score }
})
```

```ts [Enterprise: server/api/verify-recaptcha.post.ts]
export default defineEventHandler(async (event) => {
  const { token } = await readBody(event)
  const projectId = process.env.RECAPTCHA_PROJECT_ID
  const apiKey = process.env.RECAPTCHA_API_KEY
  const siteKey = process.env.NUXT_PUBLIC_SCRIPTS_GOOGLE_RECAPTCHA_SITE_KEY
  if (!projectId || !apiKey || !siteKey)
    throw createError({ statusCode: 500, message: 'Missing reCAPTCHA Enterprise configuration' })

  const response = await $fetch(
    `https://recaptchaenterprise.googleapis.com/v1/projects/${projectId}/assessments?key=${apiKey}`,
    {
      method: 'POST',
      body: {
        event: { token, siteKey, expectedAction: 'submit' },
      },
    }
  )

  const score = response.riskAnalysis?.score ?? 0
  if (!response.tokenProperties?.valid || response.tokenProperties.action !== 'submit' || score < 0.5) {
    throw createError({
      statusCode: 400,
      message: 'reCAPTCHA 验证失败',
    })
  }

  return { success: true, score }
})
```

::

::callout{type="warning"}  
切勿在客户端暴露你的密钥。请务必在服务器端验证令牌。  
::

::callout{type="info"}
令牌会在两分钟后过期。请在用户提交受保护的操作时调用 `execute`，而不是在页面加载时调用。请参阅 Google 的 [reCAPTCHA v3 放置指南](https://developers.google.com/recaptcha/docs/v3#placement_on_your_website)。
::

## 隐藏徽章

如果所需的归属声明在用户流程中仍然可见，Google [允许您隐藏 reCAPTCHA 徽章](https://developers.google.com/recaptcha/docs/faq#id-like-to-hide-the-recaptcha-badge.-what-is-allowed)：

```css
.grecaptcha-badge { visibility: hidden; }
```

```html
<p>本网站受 reCAPTCHA 保护，且受 Google 的
  <a href="https://policies.google.com/privacy">隐私政策</a> 和
  <a href="https://policies.google.com/terms">服务条款</a> 约束。
</p>
```

## 测试

对于 reCAPTCHA v3，Google 建议为[测试环境使用单独的密钥](https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do)。由于 v3 会从真实流量中学习，开发环境中的评分可能与生产环境不同。请勿在此处使用 Google 发布的始终通过验证的密钥；它们仅适用于 reCAPTCHA v2。

::script-types
::

## 示例

此示例会对联系表单提交进行评分，并在服务器上验证令牌：

::code-group

```vue [ContactForm.vue]
<script setup lang="ts">
const { onLoaded, onError } = useScriptGoogleRecaptcha()

const name = ref('')
const email = ref('')
const message = ref('')
const status = ref<'idle' | 'loading' | 'success' | 'error'>('idle')

onError(() => {
  status.value = 'error'
})

function onSubmit() {
  status.value = 'loading'

  onLoaded(({ grecaptcha }) => {
    grecaptcha.ready(async () => {
      const token = await grecaptcha.execute('YOUR_SITE_KEY', { action: 'contact' })

      const result = await $fetch('/api/contact', {
        method: 'POST',
        body: {
          token,
          name: name.value,
          email: email.value,
          message: message.value
        }
      }).catch((error) => {
        console.error('Failed to submit contact form', error)
        return null
      })

      status.value = result ? 'success' : 'error'
    })
  })
}
</script>

<template>
  <form @submit.prevent="onSubmit">
    <input v-model="name" placeholder="姓名" required>
    <input v-model="email" type="email" placeholder="邮箱" required>
    <textarea v-model="message" placeholder="留言" required />
    <button type="submit" :disabled="status === 'loading'">
      {{ status === 'loading' ? '发送中...' : '提交' }}
    </button>
    <p v-if="status === 'success'">
      消息已发送！
    </p>
    <p v-if="status === 'error'">
      发送失败，请重试。
    </p>
  </form>
</template>
```

```ts [server/api/contact.post.ts]
export default defineEventHandler(async (event) => {
  const { token, name, email, message } = await readBody(event)

  // 验证 reCAPTCHA 令牌
  const secretKey = process.env.RECAPTCHA_SECRET_KEY
  if (!secretKey)
    throw createError({ statusCode: 500, message: 'Missing reCAPTCHA secret key' })
  const verification = await $fetch('https://www.google.com/recaptcha/api/siteverify', {
    method: 'POST',
    body: new URLSearchParams({
      secret: secretKey,
      response: token,
    }),
  })

  if (!verification.success || verification.action !== 'contact' || verification.score < 0.5) {
    throw createError({
      statusCode: 400,
      message: 'reCAPTCHA 验证失败',
    })
  }

  // 处理联系表单（发送邮件、保存到数据库等）
  console.log('联系表单已提交：', { name, email, message, score: verification.score })

  return { success: true }
})
```

::
