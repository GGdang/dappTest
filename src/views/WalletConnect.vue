<script setup lang="ts">
import { ref } from 'vue'
const count = ref(0)
function say() {
  count.value++
}
</script>

<template>
  <div class="walletConnect">
    <p>{{ count }}</p>
    <button @click="say()">Wallet Connect</button>
  </div>
</template>

<style scoped></style>
