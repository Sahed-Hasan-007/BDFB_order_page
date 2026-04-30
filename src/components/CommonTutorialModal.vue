<script setup lang="ts">
defineProps<{
  videoUrl?: string;
  title?: string;
}>();

defineEmits<{
  (e: 'close'): void;
}>();
</script>

<template>
  <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm">
    <div class="bg-white dark:bg-zinc-900 rounded-xl max-w-lg w-full p-6 shadow-xl border border-zinc-200 dark:border-zinc-800">
      <div class="flex items-center justify-between mb-4">
        <h3 class="text-lg font-bold text-zinc-900 dark:text-zinc-100">{{ title || 'Tutorial' }}</h3>
        <button @click="$emit('close')" class="text-zinc-500 hover:text-zinc-700 dark:hover:text-zinc-300">
          ✕
        </button>
      </div>
      <div class="aspect-video bg-zinc-100 dark:bg-zinc-800 rounded-lg flex items-center justify-center">
        <p class="text-zinc-500 text-sm">Video URL: {{ videoUrl }}</p>
      </div>
      <div class="mt-6 flex justify-end">
        <button @click="$emit('close')" class="px-4 py-2 bg-primary-500 hover:bg-primary-600 text-white rounded-lg font-semibold text-sm transition-colors">
          Close
        </button>
      </div>
    </div>
  </div>
</template>
