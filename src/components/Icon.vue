<script setup lang="ts">
import { Icon as IconifyIcon } from '@iconify/vue';

defineProps<{
  name: string;
  class?: string;
}>();
</script>

<template>
  <IconifyIcon :icon="name" :class="class" />
</template>
