<template>
    <div class="flex flex-col items-center justify-between w-full p-4 relative">
        <Logo />
        <h1 class="header-text text-3xl font-bold flex-grow text-center mx-auto font-ultra">CHOMPI • PAL</h1>
        <DarkMode class="absolute right-0" />
    </div>
</template>
<script>

import DarkMode from './DarkMode.vue'
import LogoDark from '../assets/logos/CHOMPI_FaceLogo_White.svg'
import LogoPeach from '../assets/logos/CHOMPI_FaceLogo_Peach.svg'
import Logo from '../assets/logos/CHOMPI_FaceLogo_BlackWht.svg'
import Checkmark from '../assets/logos/checkmark.svg'

export default {
    components: {
        DarkMode,
        Logo,
        LogoDark,
        LogoPeach
    },
};
</script>