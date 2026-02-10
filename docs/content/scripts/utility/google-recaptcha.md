---
title: Google reCAPTCHA
description: 在你的 Nuxt 应用中使用 Google reCAPTCHA v3。
links:
  - label: 源码
    icon: i-simple-icons-github
    to: https://github.com/nuxt/scripts/blob/main/src/runtime/registry/google-recaptcha.ts
    size: xs
---

[Google reCAPTCHA](https://www.google.com/recaptcha/about/) 利用先进的风险分析技术保护你的网站免受垃圾信息和滥用。

Nuxt Scripts 提供了一个注册脚本组合式函数 `useScriptGoogleRecaptcha`，让你能轻松在 Nuxt 应用中集成 reCAPTCHA v3。

::callout
此集成仅支持 reCAPTCHA v3（基于得分的隐形验证）。若需使用 v2 复选框，请使用标准的 reCAPTCHA 集成方式。
::

### 全局加载

::code-group

```ts [始终启用]
export default defineNuxtConfig({
  scripts: {
    registry: {
      googleRecaptcha: {
        siteKey: 'YOUR_SITE_KEY'
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
        googleRecaptcha: {
          siteKey: 'YOUR_SITE_KEY'
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
      googleRecaptcha: true,
    }
  },
  runtimeConfig: {
    public: {
      scripts: {
        googleRecaptcha: {
          // .env 文件中
          // NUXT_PUBLIC_SCRIPTS_GOOGLE_RECAPTCHA_SITE_KEY=<your-key>
          siteKey: '',
        },
      },
    },
  },
})
```

::

## useScriptGoogleRecaptcha

`useScriptGoogleRecaptcha` 组合式函数让你可以精细控制 reCAPTCHA 在你网站上的加载时机和方式。

```ts
const { proxy } = useScriptGoogleRecaptcha({
  siteKey: 'YOUR_SITE_KEY'
})

// 执行 reCAPTCHA 并获取令牌
proxy.grecaptcha.ready(async () => {
  const token = await proxy.grecaptcha.execute('YOUR_SITE_KEY', { action: 'submit' })
  // 将令牌发送到你的服务器进行验证
})
```

请参照[注册脚本](/docs/guides/registry-scripts)指南了解更多高级用法。

### GoogleRecaptchaApi

```ts
export interface GoogleRecaptchaApi {
  grecaptcha: {
    ready: (callback: () => void) => void
    execute: (siteKey: string, options: { action: string }) => Promise<string>
    enterprise?: {
      ready: (callback: () => void) => void
      execute: (siteKey: string, options: { action: string }) => Promise<string>
    }
  }
}
```

### 配置模式

首次设置脚本时必须提供以下选项。

```ts
export const GoogleRecaptchaOptions = object({
  /**
   * 你的 reCAPTCHA 网站密钥，来自 Google reCAPTCHA 管理控制台。
   */
  siteKey: string(),
  /**
   * 使用 reCAPTCHA Enterprise 而非标准的 reCAPTCHA。
   */
  enterprise: optional(boolean()),
  /**
   * 通过 recaptcha.net 而非 google.com 加载（适用于中国）。
   */
  recaptchaNet: optional(boolean()),
  /**
   * reCAPTCHA 小部件的语言代码。
   */
  hl: optional(string()),
})
```

## 示例

使用 reCAPTCHA v3 保护表单提交，并做服务器端验证。

::code-group

```vue [ContactForm.vue]
<script setup lang="ts">
const { onLoaded } = useScriptGoogleRecaptcha()

const name = ref('')
const email = ref('')
const message = ref('')
const status = ref<'idle' | 'loading' | 'success' | 'error'>('idle')

async function onSubmit() {
  status.value = 'loading'

  onLoaded(async ({ grecaptcha }) => {
    // 获取 reCAPTCHA 令牌
    const token = await grecaptcha.execute('YOUR_SITE_KEY', { action: 'contact' })

    // 将表单数据和令牌发送到 API 进行验证
    const result = await $fetch('/api/contact', {
      method: 'POST',
      body: {
        token,
        name: name.value,
        email: email.value,
        message: message.value
      }
    }).catch(() => null)

    status.value = result ? 'success' : 'error'
  })
}
</script>

<template>
  <form @submit.prevent="onSubmit">
    <input v-model="name" placeholder="姓名" required />
    <input v-model="email" type="email" placeholder="邮箱" required />
    <textarea v-model="message" placeholder="消息" required />
    <button type="submit" :disabled="status === 'loading'">
      {{ status === 'loading' ? '发送中...' : '提交' }}
    </button>
    <p v-if="status === 'success'">消息已发送！</p>
    <p v-if="status === 'error'">发送失败。请重试。</p>
  </form>
</template>
```

```ts [server/api/contact.post.ts]
export default defineEventHandler(async (event) => {
  const { token, name, email, message } = await readBody(event)

  // 验证 reCAPTCHA 令牌
  const secretKey = process.env.RECAPTCHA_SECRET_KEY
  const verification = await $fetch('https://www.google.com/recaptcha/api/siteverify', {
    method: 'POST',
    body: new URLSearchParams({
      secret: secretKey,
      response: token,
    }),
  })

  if (!verification.success || verification.score < 0.5) {
    throw createError({
      statusCode: 400,
      message: 'reCAPTCHA 验证失败',
    })
  }

  // 处理联系表单（发送邮件、存入数据库等）
  console.log('联系表单提交信息:', { name, email, message, score: verification.score })

  return { success: true }
})
```

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

## 中国支持

需要支持中国大陆的网站，使用 `recaptchaNet: true`，从 `recaptcha.net` 而非 `google.com` 加载：

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

## 服务器端验证

reCAPTCHA 令牌必须在服务器端进行验证。请创建一个 API 接口来校验令牌：

::code-group

```ts [server/api/verify-recaptcha.post.ts]
export default defineEventHandler(async (event) => {
  const { token } = await readBody(event)
  const secretKey = process.env.RECAPTCHA_SECRET_KEY

  const response = await $fetch('https://www.google.com/recaptcha/api/siteverify', {
    method: 'POST',
    body: new URLSearchParams({
      secret: secretKey,
      response: token,
    }),
  })

  if (!response.success || response.score < 0.5) {
    throw createError({
      statusCode: 400,
      message: 'reCAPTCHA 验证失败',
    })
  }

  return { success: true, score: response.score }
})
```

```ts [企业版 - server/api/verify-recaptcha.post.ts]
export default defineEventHandler(async (event) => {
  const { token } = await readBody(event)
  const projectId = process.env.RECAPTCHA_PROJECT_ID
  const apiKey = process.env.RECAPTCHA_API_KEY
  const siteKey = process.env.NUXT_PUBLIC_SCRIPTS_GOOGLE_RECAPTCHA_SITE_KEY

  const response = await $fetch(
    `https://recaptchaenterprise.googleapis.com/v1/projects/${projectId}/assessments?key=${apiKey}`,
    {
      method: 'POST',
      body: {
        event: { token, siteKey, expectedAction: 'submit' },
      },
    }
  )

  if (!response.tokenProperties?.valid || response.riskAnalysis?.score < 0.5) {
    throw createError({
      statusCode: 400,
      message: 'reCAPTCHA 验证失败',
    })
  }

  return { success: true, score: response.riskAnalysis.score }
})
```

::

::callout{type="warning"}
切勿在客户端暴露你的密钥。请务必在服务器端验证令牌。
::

## 隐藏徽章

reCAPTCHA v3 会在网站角落显示徽章。你可以通过 CSS 将其隐藏，但必须在你的表单中包含归属声明：

```css
.grecaptcha-badge { visibility: hidden; }
```

```html
<p>本网站受 reCAPTCHA 保护，且受 Google 的
  <a href="https://policies.google.com/privacy">隐私政策</a> 和
  <a href="https://policies.google.com/terms">服务条款</a> 约束。
</p>
```

## 测试密钥

Google 提供了始终通过验证的测试密钥，适用于开发环境本地测试：

| 密钥类型 | 值 |
|----------|-------|
| 网站密钥 | `6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI` |
| 密钥秘钥 | `6LeIxAcTAAAAAGG-vFI1TnRWxMZNFuojJ4WifJWe` |

::callout{type="info"}
测试密钥总是返回 `success: true` 且得分为 `0.9`。详情参见 [Google 常见问题](https://developers.google.com/recaptcha/docs/faq#id-like-to-run-automated-tests-with-recaptcha.-what-should-i-do)。
::

```ts [nuxt.config.ts]
export default defineNuxtConfig({
  $development: {
    scripts: {
      registry: {
        googleRecaptcha: {
          siteKey: '6LeIxAcTAAAAAJcZVRqyHh71UMIEGNQ_MXjiZKhI'
        }
      }
    }
  }
})
```