<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { RouterLink } from "vue-router";
const caughtCount = ref(0);
</script>

<template>
  <div class="navbar py-3 bg-gray-100 w-full">
    <div class="flex items-center justify-center w-full">
      <router-link to="/">
        <img src="../assets/pokeLog.png" class="h-20" alt="pokemon" />
      </router-link>
    </div>
  </div>
</template>

<style>
.poke {
  width: 30px;
}
</style>