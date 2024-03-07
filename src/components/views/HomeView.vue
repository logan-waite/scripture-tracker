<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import { format, eachDayOfInterval, isEqual, isSameDay } from 'date-fns';
import { useLocalStorage } from '@vueuse/core';
import type { TrackedDay } from '@/types';

const { pagesInBom } = defineProps<{ pagesInBom: number }>();

const pagesPerDay = useLocalStorage<TrackedDay[]>('tracker', []);
const targetDate = ref(pagesPerDay.value.at(-1)?.date ?? '');

const today = new Date().toDateString();
const currentDayIndex = pagesPerDay.value.findIndex(({ date }) => isSameDay(date, today));
const startingPageNumber = computed(() =>
    currentDayIndex <= 0
        ? 0
        : pagesPerDay.value.slice(0, currentDayIndex).reduce((sum, { pages }) => {
              return sum + pages;
          }, 0)
);
const currentPage = computed(() =>
    pagesPerDay.value.slice(0, currentDayIndex + 1).reduce((sum, { pages }) => sum + pages, 0)
);
const pageInput = ref(currentPage.value);
watch(pageInput, (val) => {
    const pagesReadToday = val - startingPageNumber.value;
    pagesPerDay.value[currentDayIndex] = {
        ...pagesPerDay.value[currentDayIndex],
        pages: pagesReadToday
    };
});

const days = eachDayOfInterval({
    start: today,
    end: pagesPerDay.value.at(-1)?.date ?? today
});
const dailyPages = Math.floor((pagesInBom - startingPageNumber.value) / days.length);
const leftoverPages = (pagesInBom - startingPageNumber.value) % days.length;

const updatedSchedule = days.map((date, i) => {
    const pages = leftoverPages >= i + 1 ? dailyPages + 1 : dailyPages;
    return { date, target: pages };
});

const todaysTarget = ref(updatedSchedule.find(({ date }) => isEqual(date, today))?.target || 0);
const targetPage = ref(startingPageNumber.value + todaysTarget.value);
const pagesToReadToday = computed(
    () => todaysTarget.value - (currentPage.value - startingPageNumber.value)
);
const pagesLeft = computed(() => pagesInBom - currentPage.value);
</script>

<template>
    <div id="tracker">
        <h1>Target Date: {{ format(targetDate, 'MMMM d, y') }}</h1>
        <h1>Target Page: {{ targetPage }}</h1>
        <h1>Pages to Read Today: {{ pagesToReadToday }}</h1>
        <span>Current Page: </span><input v-model="pageInput" />
        <h1>Pages Left: {{ pagesLeft }}</h1>
    </div>
</template>
