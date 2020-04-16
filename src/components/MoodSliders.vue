<template>
    <div>
        <div div v-bind:key="sliders.name" v-for="(sliders, categoryName) in sliders">
            <MoodSliderCategory v-bind:categoryName="categoryName" v-bind:sliders="sliders"/>
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
    props: ['sliders'],
}
</script>

<style scoped>
</style>