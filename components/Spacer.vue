<template>
  <div :style="padding"></div>
</template>

<script setup lang="ts">
const props = defineProps<{
  p: number
}>()

const padding = () => {
  return {
    '--padding': `${props.p}px`
  }
}
</script>

<style scoped>
div {
  --padding: v-bind(`${props.p}px`);
  padding-top: var(--padding);
  padding-bottom: var(--padding);
}
</style>
