<template>
  <SlotDemo>
    <p>Hello World</p>
  </SlotDemo>
</template>

<script setup>
import Ref from "./components/Ref.vue";
import Computed from "./components/Computed.vue";
import Graph3D from "./components/Graph3D.vue";
import Child from "./components/Child.vue";
import { ref } from "vue";
import Dify from "./components/Dify.vue";
import SlotDemo from "./components/Slot-demo.vue";
</script>
