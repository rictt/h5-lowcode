<template>
  <!-- 单独做成一个业务组件or路由 -->
  <div class="preview">
    <component v-for="item in componentList" v-bind="{ ...item.tplData }" :key="item.name" :is="item.name"></component>
  </div>
</template>

<script>
import { onMounted, ref } from 'vue'
export default {
  setup() {
    const componentList = ref([])

    window.addEventListener("message", (message) => {
      console.log('Receive Message: ', message.data)
      const data = JSON.parse(message.data)
      componentList.value = data
    }, false);

    onMounted(() => {
      if (window.top !== window) {
        window.top.postMessage("message from child !")
      }
    })

    return {
      componentList,
    }
  }
}
</script>

<style lang="stylus" scoped>
</style>