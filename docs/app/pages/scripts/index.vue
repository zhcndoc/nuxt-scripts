<script setup lang="ts">
import { useScriptsRegistry } from '~/composables/useScriptsRegistry'

const categories = {
  ads: {
    label: '广告',
    description: '通过广告实现网站的货币化。',
  },
  analytics: {
    label: '分析',
    description: '跟踪用户及其在网站上的行为。',
  },
  tracking: {
    label: '跟踪',
    description: '更高级的用户跟踪和分析。',
  },
  marketing: {
    label: '营销',
    description: '使用实时聊天、调查等工具与用户互动。',
  },
  payments: {
    label: '支付',
    description: '在您的网站上访问支付功能。',
  },
  content: {
    label: '内容',
    description: '在您的网站上显示视频、地图和其他内容。',
  },
  support: {
    label: '支持',
    description: '实时聊天、帮助台和其他支持工具。',
  },
  utility: {
    label: '工具',
    description: '帮助您构建网站的各种工具。',
  },
}

defineOgImageComponent('Home')

useSeoMeta({
  title: '脚本注册表',
  description: '该注册表是一个包含第三方脚本的集合，为 Nuxt Scripts 提供开箱即用的可组合和组件集成。',
})

// group by category
const scriptsCategories = (await useScriptsRegistry()).reduce((acc, script) => {
  if (!acc[script.category])
    acc[script.category] = []
  acc[script.category].push({
    ...script,
    // turn title label into a camel case slug
    key: script.label.toLowerCase().replace(/ /g, '-'),
  })
  return acc
}, {})
</script>

<template>
  <div>
    <UContainer>
      <UPage>
        <UPageHeader title="脚本注册表" description="该注册表是一个包含第三方脚本的集合，为 Nuxt Scripts 提供开箱即用的可组合和组件集成。" />
        <UPageBody>
          <p class="mb-5">
            要了解有关这些脚本的更多信息，请阅读 <NuxtLink to="/docs/guides/registry-scripts" class="underline text-primary">
              注册表脚本
            </NuxtLink> 文档。
          </p>
          <div class="space-y-10">
            <div v-for="(scripts, category) in scriptsCategories" :key="category">
              <div class="mb-5">
                <h2 class="text-2xl mb-1 font-bold">
                  {{ categories[category].label }}
                </h2>
                <p class="text-gray-500 text-sm">
                  {{ categories[category].description }}
                </p>
              </div>
              <div class="flex flex-wrap gap-4">
                <NuxtLink v-for="(script, key) in scripts" :key="key" :to="`/scripts/${category}/${script.key}`">
                  <UCard class="min-w-[120px] text-center hover:bg-gray-100 dark:hover:bg-gray-800 transition">
                    <div v-if="script.logo" class="mb-2">
                      <template v-if="typeof script.logo !== 'string'">
                        <div class="logo h-10 w-auto block dark:hidden" v-html="script.logo.light" />
                        <div class="logo h-10 w-auto hidden dark:block" v-html="script.logo.dark" />
                      </template>
                      <div v-else-if="script.logo.startsWith('<svg')" class="logo h-10 w-auto" v-html="script.logo" />
                      <img v-else class="h-10 w-auto mx-auto logo" :src="script.logo">
                    </div>
                    <div class="text-gray-500 text-sm font-semibold">
                      {{ script.label }}
                    </div>
                  </UCard>
                </NuxtLink>
              </div>
            </div>
          </div>
          <div class="mt-10">
            <p>寻找缺失的集成？</p>
            <div class="mt-2">
              <UButton
                to="https://github.com/nuxt/scripts/discussions/177"
                target="_blank"
                icon="i-simple-icons-github"
                color="white"
                size="lg"
              >
                建议一个新脚本
              </UButton>
            </div>
          </div>
        </UPageBody>
      </UPage>
    </UContainer>
  </div>
</template>

<style scoped>
.logo svg {
  max-height: 100%;
  max-width: 100%;
  height: auto;
  width: auto;
  display: block;
  margin: 0 auto;
}
</style>
