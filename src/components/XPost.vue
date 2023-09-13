<script setup>
defineProps({
  address: String,
  timestamp: String,
  message: String,
})
</script>

<template>
  <div
    class="border border-gray-700 p-4 rounded mb-2"
  >
    <div class="truncate">Carteira: {{ address }}</div>
    <div>Data: {{ timestamp }}</div>
    <div>Mensagem: {{ message }}</div>
  </div>
</template>
