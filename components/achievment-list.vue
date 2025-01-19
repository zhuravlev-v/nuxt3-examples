<template>
  <div class="flex flex-col sm:pr-8 overflow-hidden">
    <p-years-range
      v-model="curentYear"
      class="flex-grow-0 hidden sm:flex"
      :min="minYear"
      :max="maxYear"
      @update:model-value="onYearChangeHandler"
    />
    <u-select
      v-model="selectedYear"
      :options="mobileYears"
      class="sm:hidden"
      @change="(e) => onYearChangeHandler(Number(e.target.value))"
    />
    <div ref="listRef" class="achievment-list w-full overflow-x-auto p-scroll pt-8 sm:pt-16">
      <div class="scroller flex gap-3 sm:gap-8 pr-8 pb-6">
        <achievment-ui-card
          v-for="item in items"
          :key="item.id"
          class="flex-shrink-0 max-w-[220px] xs:max-w-[300px]
          sm:max-w-[360px] md:max-w-[420px]  lg:max-w-[540px]"
          :data="item"
          :level="3"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { AchievmentListImpl } from '~/shared/types/achievment-list';

const props = defineProps<AchievmentListImpl>();
const selectedYear = ref(props.years[0]);
const cardSelector = '.achievment-card';
const minYear = computed(() => Math.min(...props.years));
const maxYear = computed(() => Math.max(...props.years));
const curentYear = ref(minYear.value);
const listRef = ref<HTMLDivElement>();
const observer = ref<IntersectionObserver>();

const mobileYears = computed(() => props.years.map(el => `${el}`));

const onYearChangeHandler = (value: number) => {
  const card = listRef.value?.querySelector(`${cardSelector}[data-year="${value}"]`);
  const cardX = card!.getBoundingClientRect().x;
  const listX = listRef.value?.getBoundingClientRect().x || 0;
  const { scrollLeft } = listRef.value || { scrollLeft: 0 };
  const x = cardX - listX + scrollLeft;
  listRef.value?.scrollTo({ behavior: 'smooth', left: x });
};

onMounted(() => {
  observer.value = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        curentYear.value = Number(entry.target.getAttribute('data-year'));
        selectedYear.value = curentYear.value;
      }
    });
  }, {
    threshold: 1,
    root: listRef.value,
  });

  if (listRef.value) {
    const items = listRef.value.querySelectorAll(cardSelector);
    items.forEach((item) => {
      observer.value?.observe(item);
    });

    // ставим скролл в конечную позицию
    listRef.value.scrollTo(listRef.value.scrollWidth, 0);
  }
});

onUnmounted(() => {
  if (observer.value) {
    observer.value.disconnect();
  }
});
</script>
