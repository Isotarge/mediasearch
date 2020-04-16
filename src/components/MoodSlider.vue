<template>
    <div>
        <span>{{slider.low}}</span><input name="" min='0' max='100' step='1' type='range'><span>{{slider.high}}</span>
    </div>
</template>

<script>
export default {
    name: "MoodSlider",
    props: ["slider"]
}
</script>

<style scoped>

</style>