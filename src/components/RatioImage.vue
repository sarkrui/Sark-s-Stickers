<template>
  <div class="relative rounded-lg overflow-hidden bg-gray-100" ref="container">
    <div class="pb-[100%]"></div>
    <img
      v-if="isEnter"
      :src="src"
      class="absolute inset-0 opacity-0 transition-opacity"
      @load="isLoaded = true"
      :class="{ 'opacity-100': isLoaded }"
    />
  </div>
</template>
<script lang="ts" setup>
import { ref } from "vue";
import { useIntersectionObserver } from "@vueuse/core";

const props = defineProps({
  src: {
    type: String,
    required: true,
  },
});

const isLoaded = ref(false);
const isEnter = ref(false);

const container = ref<HTMLElement>();

useIntersectionObserver(
  container,
  ([entry]) => {
    if (entry.isIntersecting) {
      isEnter.value = true;
    }
  },
  {
    rootMargin: "10%",
  }
);
</script>
