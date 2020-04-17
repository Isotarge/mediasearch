<template>
  <div>
    <h1>{{ msg }}</h1>
    <p>
      This tool will take your current feelings, desires, style preferences, and spit out some media it thinks you'll like.
      <br />It was written by Isotarge. Hope you enjoy!
    </p>
  </div>
</template>

<script>
export default {
  name: "Header",
  props: {
    msg: String
  }
};
</script>

<style scoped>
</style>
