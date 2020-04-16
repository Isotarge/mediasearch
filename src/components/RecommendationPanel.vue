<template>
  <div id="recommendationPanel">
    <input type='number' min='1' step='1' v-model="numRecommendations">
    <div>
        <Recommendation/>
    </div>
  </div>
</template>

<script>
import Recommendation from './Recommendation.vue'

export default {
  name: 'RecommendationPanel',
  components: {
    Recommendation
  },
  props: {
    numRecommendations: Number,
  }
}
</script>

<style scoped>
</style>
