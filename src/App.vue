<!-- App.vue -->
<template>
  <div id="app">
    <div id="monitor">
      <div id="crt" class="off" onClick="handleClick(event)">
        <!-- slowly moving scanline -->
        <div class="scanline"></div>
        <Header />
        <DarkMode />
        <WAVConverter />
        <Footer/>
      </div>
    </div>
  </div>
</template>

<script>
import WAVConverter from './components/WAVConverter.vue';
import DarkMode from './components/DarkMode.vue'
import Header from './components/Header.vue'
import Footer from './components/Footer.vue'

export default {
  components: {
    WAVConverter,
    Header,
    Footer
  },
};
</script>

<style>
  html.dark {
    color-scheme: dark;
  }
</style>