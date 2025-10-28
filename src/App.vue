<template>
  <div class="app">
    <Navigation />
    <router-view />
  </div>
</template>

<script setup>
import Navigation from './components/Navigation.vue'
</script>

<style lang="scss">
:root {
  --color-orange: oklch(95% .4 95);
  --color-pink: oklch(55% .45 350);
}

h1, h2, h3, h4 {
  font-family: "BBH Sans Bartle", sans-serif;
  text-wrap: balance;
  background: linear-gradient(
    to bottom left in oklab,
    var(--color-pink),
    var(--color-orange)
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  text-fill-color: transparent;
  font-weight: 900;
}

body {
  padding: 0;
  margin: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.app {
  min-height: 100vh;
  padding-top: 70px; // Space for fixed navigation
}

// Smooth scrolling
html {
  scroll-behavior: smooth;
}
</style>
