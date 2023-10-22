<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  export let slider;
  export let categoryName;
  export let defaultValue;
  
  let ignore = true;
  let value = defaultValue;

  function onChange() {
    dispatch("signalRecomputeRecommendations", {
      categoryName,
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

<form>
  <label for="ignoreCheckbox">Ignore</label>
  <input id="ignoreCheckbox" type="checkbox" bind:checked={ignore} on:change={onChange} />
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
</form>

<style>
  .moodLabel {
    min-width: 90px;
    display: inline-block;
    padding: 8px;
  }
</style>
