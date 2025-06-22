<template>
  <div class="app">
    <home-page />
  </div>
</template>

<script setup>
import { onMounted, reactive } from "vue";
import HomePage from "./components/HomePage.vue";
const state = reactive({});

onMounted(() => {});
</script>

<style scoped>
.app {
  width: 100vw;
  height: 100vh;
}
</style>
