<template>
  <div id="app">
    <Header msg="Media Search"/>
    <MoodSliders v-bind:sliders="sliders"/>
    <RecommendationPanel v-bind:numRecommendations="numRecommendations"/>
  </div>
</template>

<script>
import Header from './components/Header.vue'
import MoodSliders from './components/MoodSliders.vue'
import RecommendationPanel from './components/RecommendationPanel.vue'

export default {
  name: 'App',
  components: {
    Header,
    MoodSliders,
    RecommendationPanel
  },
  data() {
    return {
      numRecommendations: 10,
      msg: "Media Search",
      sliders: {
        Feeling: [
          {name: "happy",      low: "Sad",               high: "Happy"},
          {name: "conformist", low: "Contrarian",        high: "Conformist"},
          {name: "focused",    low: "Tired/Overwhelmed", high: "Focused"},
        ],
        Desire: [
          {name: "happiness",  low: "Sad",               high: "Happy"},
          {name: "anger",      low: "Peaceful",          high: "Angry"},
          {name: "sexy",       low: "Sexn't",            high: "Sexy"},
        ],
        Style: [
          {name: "lyrics",     low: "Instrumental",      high: "Lyrical"},
          {name: "electronic", low: "Analog",            high: "Digital"},
          {name: "melodic",    low: "Minimal",           high: "Melodic"},
          {name: "age",        low: "New",               high: "Old"},
          {name: "speed",      low: "Slow",              high: "Fast"},
        ],
      },
    }
  },
  methods: {
    computeRecommendations() {
      console.log("RECOMPUTING RECOMMENDATIONS");
    }
  }
}
</script>

<style>
</style>
