<template>
  <div>
    <h1>Child {{ msg }}</h1>
  </div>
</template>

<script setup>
import { ref } from "vue";

const props = defineProps({
  msg: {
    type: String,
  },
});
</script>
