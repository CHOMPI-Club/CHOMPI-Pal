<template>
    <div class="flex flex-col items-center justify-between w-full p-4 relative">
        <!-- Conditionally render dark and light logos -->
        <LogoDark v-if="isDark" />
        <Logo v-else />

        <!-- Header text -->
        <h1 class="header-text text-4xl font-bold flex-grow text-center mx-auto font-ultra">CHOMPI • PAL</h1>

        <!-- Dark Mode toggle button -->
        <DarkMode class="absolute right-0" />
    </div>
</template>

<script setup>
// Import components and SVGs
import DarkMode from './DarkMode.vue';
import LogoDark from '../assets/logos/CHOMPI_FaceLogo_White.svg';
import Logo from '../assets/logos/CHOMPI_FaceLogo_BlackWht.svg';
import { useDark } from "@vueuse/core";

// Declare and track the dark mode state
const isDark = useDark();
</script>
