<script setup lang="ts">
import StartForm from '@/components/StartForm.vue';
import HomeView from '@/components/views/HomeView.vue';
import { useLocalStorage } from '@vueuse/core';
import type { TrackedDay } from '@/types';

const pagesPerDay = useLocalStorage<TrackedDay[]>('tracker', []);
const pagesInBom = 531;
</script>

<template>
    <main>
        <start-form v-if="!pagesPerDay.length" :pages-in-bom="pagesInBom" />
        <home-view v-else :pages-in-bom="pagesInBom" />
    </main>
</template>

<style scoped></style>
