<script setup lang="ts">
import { withBase } from "vitepress";

defineProps<{
  href?: string;
  target?: string;
  ariaLabel?: string;
  rel?: string;
}>();

const emit = defineEmits<{
  click: [e: MouseEvent];
}>();
</script>

<template>
  <a :rel :href="href ? withBase(href) : undefined" :target :aria-label @click="e => emit('click', e)">
    <slot></slot>
  </a>
</template>
