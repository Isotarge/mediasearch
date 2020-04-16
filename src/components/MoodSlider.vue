<template>
    <div>
        Ignore
        <input type='checkbox' v-model="ignore" @change="onChange" >
        <span>{{slider.low}}</span>
        <input type='range' min='0' max='100' step='1' v-model="value" @change="onChange">
        <span>{{slider.high}}</span>
    </div>
</template>

<script>
export default {
    name: "MoodSlider",
    props: {
        category: String,
        slider: Object,
        value: Number,
        ignore: Boolean
    },
    methods: {
        onChange: function() {
            this.$emit('signal-recompute-recommendations', this.category, this.slider.name, parseInt(this.value), this.ignore);
        }
    }
}
</script>

<style scoped>

</style>