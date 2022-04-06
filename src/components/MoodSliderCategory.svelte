<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();

  import MoodSlider from "./MoodSlider.svelte";

  export let categoryName;
  export let sliders;

  function signalRecompute(e) {
    dispatch("signalRecomputeRecommendations", {
      ...e.detail,
      categoryName,
    });
  }
</script>

<h2>{categoryName.charAt(0).toUpperCase() + categoryName.slice(1)}</h2>
{#each sliders as slider}
  <MoodSlider {slider} on:signalRecomputeRecommendations={signalRecompute} />
{/each}
