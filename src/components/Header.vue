<template>
    <div class="flex flex-row items-center justify-between w-full p-4 relative">
        <h1 class="header-text text-3xl font-bold flex-grow text-center mx-auto">chompi-pal</h1>
        <DarkMode class="absolute right-0" />
    </div>
</template>
<script>
import DarkMode from './DarkMode.vue'
import Logo from '../assets/ChompiLogo.png'

export default {
    components: {
        DarkMode,
        Logo
    },
};
</script>