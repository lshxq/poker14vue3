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
    that.size = {
      height,
      width: Math.floor(height * .618)
    }
  }
}
</script>

<style scoped>
.home {
  height: 100vh;
}
</style>