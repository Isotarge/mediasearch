<template>
    <div id="moodSliders">
        <div v-bind:key="sliderCategory.name" v-for="(sliderCategory, categoryName) in sliders">
            <MoodSliderCategory v-bind:categoryName="categoryName" v-bind:sliders="sliderCategory"/>
        </div>
    </div>
</template>

<script>
import MoodSliderCategory from './MoodSliderCategory.vue'
export default {
    name: "MoodSliders",
    components: {
        MoodSliderCategory
    },
    props: [
        'sliders'
    ],
}
</script>

<style scoped>
</style>