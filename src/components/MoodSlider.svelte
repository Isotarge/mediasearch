<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  let value = 50;
  let ignore = true;
  export let slider;

  function onChange() {
    dispatch("signalRecomputeRecommendations", {
      name: slider.name,
      value,
      ignore,
    });
  }

  function onChangeSlider() {
    ignore = false;
    onChange();
  }
</script>

<div>
  Ignore
  <input type="checkbox" bind:checked={ignore} on:change={onChange} />
  <span class="moodLabel">{slider.low}</span>
  <input
    type="range"
    min="0"
    max="100"
    step="1"
    bind:value
    on:change={onChangeSlider}
  />
  <span>{value}</span>
  <span class="moodLabel">{slider.high}</span>
</div>

<style>
  .moodLabel {
    min-width: 90px;
    display: inline-block;
    padding: 8px;
  }
</style>
