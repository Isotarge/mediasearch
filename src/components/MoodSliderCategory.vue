<template>
    <div>
        <h2>{{ categoryName }}</h2>
        <div div v-bind:key="slider.name" v-for="slider in sliders">
            <MoodSlider v-bind:category="categoryName" v-bind:slider="slider"/>
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
    props: [
        'categoryName',
        'sliders'
    ],
}
</script>

<style scoped>
</style>