<template>
  <div id="app">
    <Header msg="Media Search" />
    <div id="panel-container">
      <MoodSliders :sliders="sliders" @signal-recompute-recommendations="computeRecommendations" />
      <RecommendationPanel
        ref="RMPanel"
        :numRecommendations="numRecommendations"
        :MusicRecommendations="MusicRecommendations"
      />
    </div>
  </div>
</template>

<script>
import Header from "./components/Header.vue";
import MoodSliders from "./components/MoodSliders.vue";
import RecommendationPanel from "./components/RecommendationPanel.vue";
import MusicRecommendations from "./MusicRecommendations.json";

export default {
  name: "App",
  components: {
    Header,
    MoodSliders,
    RecommendationPanel
  },
  data() {
    return {
      numRecommendations: 10,
      MusicRecommendations: MusicRecommendations,
      msg: "Media Search",
      // TODO: Move this to external JSON
      sliders: {
        feeling: [
          { name: "happy", low: "Sad", high: "Happy" },
          { name: "conformist", low: "Contrarian", high: "Conformist" },
          { name: "focused", low: "Tired", high: "Focused" }
        ],
        desire: [
          { name: "happiness", low: "Sad", high: "Happy" },
          { name: "anger", low: "Peaceful", high: "Angry" },
          { name: "sexy", low: "Sexn't", high: "Sexy" }
        ],
        style: [
          { name: "lyrics", low: "Instrumental", high: "Lyrical" },
          { name: "electronic", low: "Analog", high: "Digital" },
          { name: "melodic", low: "Minimal", high: "Melodic" },
          { name: "age", low: "New", high: "Old" },
          { name: "speed", low: "Slow", high: "Fast" }
        ]
      }
    };
  },
  methods: {
    computeRecommendations(categoryName, sliderName, sliderValue, ignore) {
      this.$refs.RMPanel.updateSearchSetting(
        categoryName,
        sliderName,
        sliderValue,
        ignore
      );
    }
  }
};
</script>

<style>
body {
  font-family: Arial, Helvetica, sans-serif;
}
textarea {
  font-family:'Courier New', Courier, monospace;
}
#panel-container {
  display: grid;
  grid-template-columns: 50% 50%;
}
</style>
