<template>
  <div class="mx-auto px-10 pt-10 lg:w-1/2">
    <p class="mb-2 text-center text-5xl font-bold tracking-tight">
      Lukaja's Nuxt Template
    </p>
    <p class="mb-4 text-center">
      Nuxt Template used for my personal projects. Includes NuxtUI, ESLint,
      Prettier, and Google Analytics.
    </p>
    <p class="mb-2 mt-4 text-xl font-semibold">Installation</p>
    <p>
      1. Clone
      <a
        href="https://github.com/lukajaa/lukajaa-nuxt-template/"
        target="_blank"
        class="font-semibold text-blue-400 hover:text-blue-500"
      >
        this repo
      </a>
    </p>
    <p>2.</p>
    <UAlert title="Terminal" icon="i-heroicons-command-line" class="text-left">
      <template #description>yarn <br />yarn dev </template>
    </UAlert>
    <p class="mb-2 mt-4 text-xl font-semibold">Updating</p>
    <UAlert
      icon="i-heroicons-command-line"
      title="Terminal"
      class="mb-4 text-left"
    >
      <template #description>
        yarn outdated<br />yarn add -D packages
      </template>
    </UAlert>

    <p class="mb-2 mt-4 text-xl font-semibold">Dependencies</p>
    <p v-for="(version, dependency) of devDependencies" :key="dependency">
      {{ dependency }}: {{ version.replace('^', '') }}
    </p>
    <ClientOnly>
      <UButton
        :icon="
          isDark ? 'i-heroicons-moon-20-solid' : 'i-heroicons-sun-20-solid'
        "
        color="gray"
        class="fixed right-0 top-0 m-4"
        size="xl"
        @click="isDark = !isDark"
      />
      <template #fallback>
        <div class="h-8 w-8" />
      </template>
    </ClientOnly>
  </div>
</template>

<script setup lang="ts">
const colorMode = useColorMode();
const isDark = computed({
  get() {
    return colorMode.value === 'dark';
  },
  set() {
    colorMode.preference = colorMode.value === 'dark' ? 'light' : 'dark';
  },
});

const devDependencies = {
  '@nuxt/devtools': '^1.0.0',
  '@nuxt/eslint-config': '^0.2.0',
  '@nuxt/ui': '^2.9.0',
  '@nuxtjs/eslint-config-typescript': '^12.1.0',
  '@nuxtjs/eslint-module': '^4.1.0',
  '@tailwindcss/typography': '^0.5.10',
  '@types/node': '^20.8.9',
  '@typescript-eslint/parser': '^6.9.1',
  eslint: '^8.52.0',
  'eslint-config-prettier': '^9.0.0',
  'eslint-loader': '^4.0.2',
  'eslint-plugin-prettier': '^5.0.1',
  'eslint-plugin-vue': '^9.18.1',
  nuxt: '^3.8.0',
  'nuxt-gtag': '^1.1.1',
  prettier: '^3.0.3',
  'prettier-plugin-tailwindcss': '^0.5.6',
  sass: '^1.69.5',
  typescript: '^5.2.2',
};
</script>
