<!-- App.vue -->
<template>
  <div id="app">
    <Header />
    <DarkMode />
    <WAVConverter />
    <Footer />
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