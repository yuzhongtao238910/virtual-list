<script setup>
import VirtualList from "./components/virtual-list.vue"
import Item from "./components/Item.vue"
import Mock from "mockjs"
import { ref } from "vue"
const items = ref([])
// for (let i = 0; i < 100; i++) {
//   items.value.push({
//     id: i,
//     value: i
//   })
// }

for (let i = 0; i < 1000; i++) {
  items.value.push({
    id: i,
    value: Mock.Random.sentence(5, 50)
  })
}

// size:告诉我每一项多高，因为需要算出来一个滚动条


// variable 代表并不知道每一行的高度，但是在初始化阶段仍然要给一个自己认为的大致的高度
// 此时会缓存每一个line的高度

// variable表示这个高度不一定有多高

</script>


<template>
  <VirtualList
    :size="100"
    :remain="8"
    :items="items"
    :variable="true"
  >
    <Item>
    </Item>
  </VirtualList>
</template>

<style scoped>

</style>
