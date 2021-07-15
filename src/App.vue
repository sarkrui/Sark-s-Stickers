<template>
  <div class="min-h-screen flex flex-col">
    <nav>
      <div class="bg-yellow-200 pt-6 px-5">
        <img src="./assets/heading.png" class="w-40 md:w-72 mx-auto" />
      </div>
    </nav>
    <div class="sticky top-0 bg-yellow-200 px-5 py-5 z-10">
      <div class="relative max-w-4xl mx-auto">
        <input
          v-model="searchStr"
          class="
            border-2 border-black
            rounded-full
            bg-white
            !outline-none
            w-full
            pl-5
            pr-8
            py-1
            focus:shadow-md
            transition
          "
          placeholder="通过名称或标签搜索表情"
        />
        <SearchIcon
          class="absolute right-3 h-1/2 top-1/2 -translate-y-1/2"
        ></SearchIcon>
      </div>
    </div>
    <main class="flex-grow bg-white px-5">
      <div class="max-w-4xl mx-auto" ref="mainInner">
        <div class="text-gray-500 mt-5 text-center" v-if="!displayList.length">
          没有找到你需要的表情呢~
        </div>
        <div
          v-for="item in displayList"
          :key="item.label"
          class="py-8 space-y-8"
        >
          <label class="category-label">{{ item.label }}</label>
          <div class="category-container">
            <div
              v-for="child in item.children"
              :key="child.name"
              class="relative"
            >
              <RatioImage :src="child.url"></RatioImage>
              <p class="break-all text-center mt-5">{{ child.name }}</p>
              <div
                class="
                  text-xs
                  mt-2
                  text-center
                  flex flex-wrap
                  justify-center
                  space-x-1
                "
              >
                <span
                  v-for="tag in child.tags"
                  class="bg-blue-100 px-2 py-1 rounded mb-1"
                >
                  {{ tag }}</span
                >
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
    <footer class="text-center py-3 text-gray-500">Copyright @Sark</footer>
  </div>
</template>

<script lang="ts" setup>
import { computed, onMounted, onUnmounted, ref, watch } from "vue";
import SearchIcon from "./components/SearchIcon.vue";
import RatioImage from "./components/RatioImage.vue";
import Fuse from "fuse.js";
import ResizeObserver from "resize-observer-polyfill";

interface Sticker {
  name: string;
  category: string;
  tags: string[];
  url: string;
}

const categories = new Set<string>();
const stickers: Sticker[] = [];
const source = import.meta.globEager("/stickers/**/*.gif");
for (const key in source) {
  const ns = key.split("/");
  console.log(ns);
  const category = ns[2];
  const tags = [...ns[3].matchAll(/\[(.+?)\]/g)].map((item) => item[1]);
  const name = ns[3].replace(/\[(.+?)\]/g, "").replace(".gif", "");
  categories.add(category);
  stickers.push({
    name,
    category,
    tags,
    url: source[key].default,
  });
}

const fuzeKeys = ["name", "tags"];

const fuse = new Fuse(
  stickers,
  {
    shouldSort: true,
    keys: fuzeKeys,
  },
  Fuse.createIndex(fuzeKeys, stickers)
);

const searchStr = ref("");
const result = ref<Sticker[]>([...stickers]);

watch(searchStr, (val) => {
  if (val) {
    result.value = fuse.search(val).map(({ item }) => item);
  } else {
    result.value = [...stickers];
  }
});

const displayList = computed(() =>
  [...categories]
    .map((label) => ({
      label,
      children: result.value.filter((item) => item.category === label),
    }))
    .filter((item) => item.children.length)
);

const STICKER_WIDTH = 180;
const stickerCountPerLine = ref(0);
const mainInner = ref<HTMLElement>();
const resizeObserver = ref<ResizeObserver | null>(null);

onMounted(() => {
  const observer = new ResizeObserver(([entry]) => {
    stickerCountPerLine.value = Math.ceil(
      entry.contentRect.width / STICKER_WIDTH
    );
  });

  if (mainInner.value) {
    observer.observe(mainInner.value);
  }

  resizeObserver.value = observer;
});

onUnmounted(() => {
  if (resizeObserver.value) {
    resizeObserver.value.disconnect();
  }
});
</script>

<style scoped lang="postcss">
.category-label {
  @apply relative text-lg font-medium;
}
.category-label::after {
  content: "";
  @apply h-1 bg-yellow-200 absolute -bottom-2 left-0;
  width: calc(100% + 0.5rem);
}

.category-container {
  @apply grid gap-5;
  grid-template-columns: repeat(v-bind(stickerCountPerLine), minmax(0, 1fr));
}
</style>
