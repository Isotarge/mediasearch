<template>
  <div id="moodSliders">
    <div v-bind:key="sliderCategory.name" v-for="(sliderCategory, categoryName) in sliders">
      <MoodSliderCategory
        :categoryName="categoryName"
        :sliders="sliderCategory"
        @signal-recompute-recommendations="signalRecompute"
      />
    </div>
  </div>
</template>

<script>
import MoodSliderCategory from "./MoodSliderCategory.vue";
export default {
  name: "MoodSliders",
  components: {
    MoodSliderCategory
  },
  props: {
    sliders: Object
  },
  methods: {
    signalRecompute(categoryName, sliderName, sliderValue, ignore) {
      this.$emit(
        "signal-recompute-recommendations",
        categoryName,
        sliderName,
        sliderValue,
        ignore
      );
    }
  }
};
</script>

<style scoped>
</style>