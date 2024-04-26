<template>
  <div>
  	<slot></slot>
  </div>
</template>
<script setup>
import {ref, onMounted } from "vue"
const props = defineProps({
  item: Object
})
onMounted(() => {
  console.log(props.item)
})
</script>