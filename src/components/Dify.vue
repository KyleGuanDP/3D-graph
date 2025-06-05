<template>
  <div>
    <h1>Dify</h1>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { onMounted } from "vue";

const dify = ref(null);

const request = function (query) {
  fetch("https://api.dify.ai/v1/chat-messages", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${import.meta.env.VITE_DIFY_API_KEY}`,
    },
    body: JSON.stringify({ query }),
  });
};

const response = request.onMounted(() => {
  dify.value = new Dify();
});
</script>

<style scoped>
/* Your component styles here */
</style>
