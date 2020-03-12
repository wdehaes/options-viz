<template>
  <div class="container">
    <graph :input="twitter" />
  </div>
</template>

<script>
import Graph from '~/components/Graph.vue'
import * as twitterData from '~/assets/data/twitter.js'
export default {
  components: {
    Graph
  },
  data() {
    return {
      twitter: twitterData
    }
  }
}
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  flex-direction: column;
  align-items: center;
  text-align: center;
}
</style>
