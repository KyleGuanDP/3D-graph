<template>
    <div>
      <p>raw: {{ raw }}</p>
      <p>computed value: {{ value }}</p>
    </div>
  </template>
  
  <script setup>
  import { ref, computed } from 'vue'
  
  const raw = ref(2)
  
  const value = computed(() => raw.value * 2)
  </script>
    