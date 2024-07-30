<template>
  <span class="underline underline-offset-6" :class="{
    'underline-red-400 text-red-400 font-bold text-lg': important,
    'underline-dotted': !important
  }">
    <slot></slot>
  </span>
</template>

<script setup lang="ts">
withDefaults(defineProps<{
  important: boolean
}>(), {
  important: false
})
</script>