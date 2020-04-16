<template>
    <div>
        <h2>{{ categoryName.charAt(0).toUpperCase() + categoryName.slice(1) }}</h2>
        <div div v-bind:key="slider.name" v-for="slider in sliders">
            <MoodSlider v-bind:slider="slider" v-on:signal-recompute-recommendations="signalRecompute"/>
        </div>
    </div>
</template>

<script>
import MoodSlider from './MoodSlider.vue'
export default {
    name: "MoodSliderCategory",
    components: {
        MoodSlider
    },
    props: {
        categoryName: String,
        sliders: Object
    },
    methods: {
        signalRecompute(sliderName, sliderValue, ignore) {
            this.$emit('signal-recompute-recommendations', this.categoryName, sliderName, sliderValue, ignore);
        }
    }
}
</script>

<style scoped>
</style>