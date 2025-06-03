<template>
    <div>
        <button @click="handleClick">Click me</button>
        <p>{{ value }}</p>
    </div>
</template>

<script setup>
    import { ref } from 'vue'
    const value = ref(0)
    function handleClick() {
        value.value++
    }
</script>