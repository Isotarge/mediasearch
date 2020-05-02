<template>
  <div>
    Ignore
    <input type="checkbox" v-model="ignore" @change="onChange" />
    <span class="moodLabel">{{slider.low}}</span>
    <input type="range" min="0" max="100" step="1" v-model.number="value" @change="onChangeSlider" />
    <span>{{value}}</span>
    <span class="moodLabel">{{slider.high}}</span>
  </div>
</template>

<script>
export default {
  name: "MoodSlider",
  data: function() {
    return {
      value: 50,
      ignore: true,
    };
  },
  props: {
    slider: Object,
  },
  methods: {
    onChange() {
      this.$emit(
        "signal-recompute-recommendations",
        this.slider.name,
        this.value,
        this.ignore
      );
    },
    onChangeSlider() {
      this.ignore = false;
      this.onChange();
    }
  }
};
</script>

<style scoped>
.moodLabel {
  min-width: 90px;
  display: inline-block;
  padding: 8px;
}
</style>