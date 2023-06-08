<template>
  <div class="home">
    <poker-comp />
  </div>
</template>

<script>
// @ is an alias to /src
import PokerComp from '@/components/poker-comp.vue'

export default {
  name: 'HomeView',
  components: {
    PokerComp
  }
}
</script>

<style scoped>
.home {
  width: 1200px;
  height: 1500px;
  zoom: .5;
}
</style>