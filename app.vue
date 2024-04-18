<script setup lang="ts">
import { appName } from '~/constants'

useHead({
  title: appName,
})
const quasar = useQuasar()

const isDark = useDark()
watchImmediate(isDark, (isDark) => {
  quasar.dark.set(isDark)
})
</script>

<template>
  <VitePwaManifest />
  <NuxtLayout>
    <q-layout view="hHr LpR lFf">
      <NuxtPage />
    </q-layout>
  </NuxtLayout>
</template>

<style>
html,
body,
#__nuxt {
  height: 100vh;
  margin: 0;
  padding: 0;
}

html.dark {
  background: #222;
  color: white;
}
</style>
