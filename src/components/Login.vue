<script setup>
import { ref } from "@vue/reactivity"
import { ElInput } from "element-plus"


defineProps({
  msg: {
    type: String,
    required: true
  }
})
const name = ref("")
const pass = ref("")
</script>

<template>
    <ElInput v-model="name" type="text"/>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
