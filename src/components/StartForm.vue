<script setup lang="ts">
import { ref } from 'vue';
import { eachDayOfInterval } from 'date-fns';
import { useLocalStorage } from '@vueuse/core';
import type { TrackedDay } from '@/types.ts';

const pagesPerDay = useLocalStorage<TrackedDay[]>('tracker', []);
const { pagesInBom } = defineProps<{ pagesInBom: number }>();

const startDate = ref('');
const endDate = ref('');

function startTracker() {
    const days = eachDayOfInterval({
        start: startDate.value.replace(/-/g, '/'),
        end: endDate.value.replace(/-/g, '/')
    });
    const dailyPages = Math.floor(pagesInBom / days.length);
    const leftoverPages = pagesInBom % days.length;

    const trackedDays = days.map((date, i): TrackedDay => {
        const pages = leftoverPages >= i + 1 ? dailyPages + 1 : dailyPages;
        return { date: date.toDateString(), pages, target: 0 };
    });
    pagesPerDay.value = trackedDays;
}
</script>
<template>
    <form class="flex flex-col max-w-32" id="start" @submit.prevent="startTracker">
        <label for="start-date">Start Date</label>
        <input id="start-date" v-model="startDate" type="date" />
        <label for="end-date">End Date</label>
        <input id="end-date" v-model="endDate" type="date" />
        <button class="rounded-md bg-blue-600 text-white py-1 px-2 fit-content">Start</button>
    </form>
</template>
