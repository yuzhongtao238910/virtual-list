<script setup>
import throttle from "lodash/throttle"
import {ref, onMounted, computed, reactive, onUpdated, nextTick} from "vue"
const props = defineProps({
  // 此时就需要缓存每一项的高度
  variable: Boolean, // 此时为true的话代表每一项的高度是不确定的
  size: Number, // 当前每一项的高度
  remain: Number, // 可见是多少个
  items: Array // 一共是多少个
})
const itemsRef = ref(null)

const viewport = ref(null)
const scrollBar = ref(null)


// console.log(props.remain, props.items)

const offset = ref(0)
const data = reactive({
  start: 0,
  end: props.remain // 默认显示
})



// 渲染3屏幕 前后都是8
const prevCount = computed(() => {
  return Math.min(data.start, props.remain)
})
const nextCount = computed(() => {
  return Math.min(props.remain, props.items.length - data.end)
})
// 可见的数据有哪些
const visibleData = computed(() => {
  let start = data.start -  prevCount.value
  let end = data.end + nextCount.value
  return props.items.slice(start, end)
})



onMounted(() => {
  viewport.value.style.height = props.size * props.remain + 'px'
  scrollBar.value.style.height = props.size * props.items.length + 'px'
  // 320 40000 39680
  // console.log(viewport.value.style.height, scrollBar.value.style.height)


  // 如果加载完毕，需要缓存每一项的高度
  // 先计算好，等一会滚动的时候 去渲染页面的时候获取真实的dom的高度，来更新的缓存的高度
  cacheList()


  // 再重新计算滚动条的高度
})

// 实际的位置信息
const positions = ref([])

const cacheList = () => {
  // 缓存当前项的高度和top值，bottom值
  positions.value = props.items.map((item, index) => ({
    // size此时是预留的高度
    top: index * props.size,
    height: props.size,
    bottom: (index+1) * props.size
  }))
}

onUpdated(() => {
  console.log('updated', itemsRef.value)
  // 页面渲染完成后，需要根据当前展示的数据更新缓存positions
  nextTick(() => {
    // 确保一定拿到真实的dom

    
    if (itemsRef.value && itemsRef.value.length > 0) {
       itemsRef.value.forEach(node => {
         let {height} = node.getBoundingClientRect()
         // height 当前真实的高度
         // 我们希望更新老的高度
         let id = +node.getAttribute('vid')
         // console.log(id)
         let oldHeight = positions.value[id].height
         let val = oldHeight - height // 计算当前的高度是否和之前的有变化
         if (val) {
          positions.value[id].height = height
          positions.value[id].bottom = positions.value[id].bottom - val // 底部增加了

          // 链表，自己变了，后面的都需要变化 将后续的所有人，都需要向后排
          for (let i = id + 1; i < positions.value.length; i++) {
            positions.value[i].top = positions.value[i-1].bottom
            positions.value[i].bottom = positions.value[i].bottom - val
          }
         }
       })
       // 只要更新过，会算出来滚动条的最新高度
       // 就是动态的计算滚动条的高度
       scrollBar.value.style.height = positions.value[positions.value.length-1].bottom + 'px'
    }
    // 根据当前显示 更新缓存之中的height top bottom 最终更新滚动条的高度
  })
})



const getStartIndex = (value) => { // 查找当前滚动的需要找到的值
  let start = 0 // 开始
  let end = positions.value.length - 1 // 结束
  let temp = null

  while(start <= end) {
    let middleIndex = parseInt((start + end) / 2)
    let middleVal = positions.value[middleIndex].bottom // 找到当前中间 的 结束

    if (middleVal == value) { // 如果直接找到了，就返回当前的下一个位置
      return middleIndex + 1
    } else if (middleVal < value) { // 在右边
      start = middleIndex + 1
    } else if (middleVal > value) { // 在左边
      if (temp == null || temp > middleIndex) {
        temp = middleIndex // 找到范围
      }
      end = middleIndex - 1
    }
  }
  return temp
}

const handleScroll = (evt) => {
  // 应该先算出来当前的滚过去几个，当前应该从第几个开始显示
  let scrollTop = viewport.value.scrollTop


  if (props.variable) {
    // 如果有传递的variable 说明要使用二分查找找到对应的记录
    // 10000 个小朋友 使用二分查找最快被
    // 当前的列表默认就是有序的排列啊 高度 top 都是从高到低的

    data.start = getStartIndex(scrollTop)
    // console.log(data.start, 103)
    data.end = data.start + props.remain


    offset.value = positions.value[data.start - prevCount.value] 
      ? positions.value[data.start - prevCount.value].top
      : 0

    
    
  } else {
    // console.log(scrollTop)

    data.start = Math.floor(scrollTop / props.size)

    data.end = data.start + props.remain

    // 定位当前的可视区域 让可视区域滚动条调整偏移量 减去预留的渲染的位置
    offset.value = data.start * props.size - props.size * prevCount.value
  }
  
}
const handleScrollFn = throttle(handleScroll, 100, {
  leading: false
})
</script>

<template>
<!-- 可以滚动的盒子 -->
 <div class="viewport" ref="viewport" @scroll="handleScrollFn">
<!--  自己做一个滚动条 -->
    <div class="scroll-bar" ref="scrollBar"></div>
<!--  真实渲染的内容 -->
    <div class="scroll-list" :style="{transform: `translate3d(0,${offset}px,0)`}">
      <div v-for="item in visibleData" :vid="item.id" :key="item.id" ref="itemsRef">
      <!--  之前的知道有多高 -->
        <!--  <div style="height: 40px; border: 1px solid #ccc">{{item}}</div> -->

      <!--  此时的不知道有多高 -->
        <div style="padding: 20px 0; border: 1px solid #ccc">{{item}}</div>
      </div>
    </div>
 </div>
</template>

<style scoped lang="scss">
.viewport {
  overflow-y: scroll;
  position: relative;
}
.scroll-list {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
</style>
