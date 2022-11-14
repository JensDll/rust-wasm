<script setup lang="ts">
import init, { greet } from 'wasm-simple'

const handleClick = async () => {
  await init()
  greet('World')
}
</script>

<template>
  <h1>Hello Wasm</h1>
  <button @click="handleClick">Let's Go!</button>
</template>

<style scoped></style>
