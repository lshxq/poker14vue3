<template>
  <div class="home" ref="homeRef">
    <poker-comp v-if="size" :size="size"/>
  </div>
</template>

<script>
import PokerComp from '@/components/poker-comp.vue'

export default {
  name: 'HomeView',
  data() {
    return {
      size: false
    }
  },
  components: {
    PokerComp
  },
  mounted() {
    const that = this
    const height = that.$refs.homeRef.offsetHeight
    const width = that.$refs.homeRef.offsetWidth
    that.size = {
      height,
      width
    }
  }
}
</script>

<style scoped>
.home {
  height: calc(100vh);
  width: calc(100vw);
  max-width: calc(100vh * 0.618);
}
</style>