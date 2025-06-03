<!-- LazyImage.vue -->
<template>
    <img
      ref="imageRef"
      :alt="alt"
      :class="{ loaded: isLoaded }"
      :src="isLoaded ? src : placeholder"
    />
  </template>
  
  <script setup>
  import { ref, onMounted, onBeforeUnmount } from 'vue'
  
  const props = defineProps({
    src: { type: String, required: true },
    alt: { type: String, default: '' },
    placeholder: { type: String, default: 'https://via.placeholder.com/300x200?text=Loading...' }
  })
  
  const imageRef = ref(null)
  const isLoaded = ref(false)
  let observer = null
  
  onMounted(() => {
    observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          isLoaded.value = true
          observer.unobserve(entry.target)
        }
      })
    }, {
      threshold: 0.1 // 当有 10% 进入视口就触发
    })
  
    if (imageRef.value) {
      observer.observe(imageRef.value)
    }
  })
  
  onBeforeUnmount(() => {
    if (imageRef.value && observer) {
      observer.unobserve(imageRef.value)
    }
  })
  </script>
  
  <style scoped>
  img {
    width: 100%;
    height: auto;
    opacity: 0;
    transition: opacity 0.5s ease-in-out;
  }
  img.loaded {
    opacity: 1;
  }
  </style>
  