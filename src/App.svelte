<script>
  import { onMount } from "svelte";
  import MoodSlider from "./components/MoodSlider.svelte";
  import Music from "./Music.json";
  import TV from "./TV.json";
  import YouTube from "./YouTube.json";
  import Podcast from "./Podcast.json";

  let mediaType = "Podcast";
  const media = {
    Music,
    TV,
    YouTube,
    Podcast,
  };
  let pageNumber = 0;
  const recommendationsPerPage = 10;
  let matches = [];
  const currentSearchSettings = {};

  // Pagination
  $: lastPage = Math.floor(
    media[mediaType].recommendations.length / recommendationsPerPage
  );
  $: paginatedMatches = matches.slice(
    pageNumber * recommendationsPerPage,
    (pageNumber + 1) * recommendationsPerPage
  );

  function gotoFirstPage() {
    pageNumber = 0;
  }
  function gotoPreviousPage() {
    pageNumber = Math.max(0, pageNumber - 1);
  }
  function gotoNextPage() {
    pageNumber = Math.min(lastPage, pageNumber + 1);
  }
  function gotoLastPage() {
    pageNumber = lastPage;
  }

  function updateSearchSetting(e) {
    const {
      detail: { categoryName, name, value, ignore },
    } = e;

    if (!(mediaType in currentSearchSettings)) {
      currentSearchSettings[mediaType] = {};
    }
    if (!(categoryName in currentSearchSettings[mediaType])) {
      currentSearchSettings[mediaType][categoryName] = {};
    }
    currentSearchSettings[mediaType][categoryName][name] = {
      value,
      ignore,
    };
    searchMedia();
  }

  function isEnabled(categoryName, sliderName) {
    return (
      !currentSearchSettings?.[mediaType]?.[categoryName]?.[sliderName]
        ?.ignore ?? false
    );
  }

  function getDefaultValue(categoryName, sliderName) {
    return (
      currentSearchSettings?.[mediaType]?.[categoryName]?.[sliderName]
        ?.value ?? 50
    );
  }

  function checkSingleStat(media, categoryName, sliderName) {
    if (categoryName in media && sliderName in media[categoryName]) {
      const difference =
        currentSearchSettings[mediaType][categoryName][sliderName].value -
        media[categoryName][sliderName];
      return Math.abs(difference);
    }
    // Note: Tweaking this value will probably be an ongoing effort
    return 10;
  }

  function computeMediaScore(media) {
    let totalDifference = 0;
    let numEnabledSliders = 0;
    for (const categoryName in currentSearchSettings[mediaType]) {
      for (const sliderName in currentSearchSettings[mediaType][categoryName]) {
        if (isEnabled(categoryName, sliderName)) {
          numEnabledSliders++;
          totalDifference += checkSingleStat(media, categoryName, sliderName);
        }
      }
    }
    if (numEnabledSliders > 0) {
      // Average difference
      return totalDifference / numEnabledSliders;
    }
    return 0;
  }

  function searchMedia() {
    // Compute the new matches
    matches = media[mediaType].recommendations
      .map((media) => {
        return {
          media: media,
          score: computeMediaScore(media),
        };
      })
      .sort((a, b) => a.score - b.score);
  }
  onMount(searchMedia);

  // Editing
  let editMode = false;
  let newName = "";
  let newLink = "";

  function addRecommendation() {
    const newRecommendation = { name: newName };
    if (newLink.length > 0) {
      newRecommendation.link = newLink;
    }
    for (const categoryName in currentSearchSettings[mediaType]) {
      for (const sliderName in currentSearchSettings[mediaType][categoryName]) {
        if (isEnabled(categoryName, sliderName)) {
          if (!(categoryName in newRecommendation)) {
            newRecommendation[categoryName] = {};
          }
          newRecommendation[categoryName][sliderName] =
            currentSearchSettings[mediaType][categoryName][sliderName].value;
        }
      }
    }
    console.log(newRecommendation);
    media[mediaType].recommendations.push(newRecommendation);
  }

  function exportJSON() {
    function download(content, fileName, contentType) {
      const a = document.createElement("a");
      const file = new Blob([content], { type: contentType });
      a.href = URL.createObjectURL(file);
      a.download = fileName;
      a.click();
    }
    download(
      JSON.stringify(media[mediaType], null, "\t"),
      `${mediaType}.json`,
      "text/plain"
    );
  }
</script>

<header>
  <h1>Media Search</h1>
  <p>
    This tool will take your feelings, desires, and style preferences, and
    recommend some media it thinks you'll like.
    <br />It was written by Isotarge as an exercise in learning Svelte. Hope you
    enjoy!
  </p>
</header>
<main id="panel-container">
  <div id="slider-container">
    <select
      id="mediaTypeDropdown"
      bind:value={mediaType}
      on:change={searchMedia}
    >
      {#each Object.keys(media) as _media}
        <option>{_media}</option>
      {/each}
    </select>
    <label for="editMode">Edit Mode</label>
    <input id="editMode" type="checkbox" bind:checked={editMode} />
    {#if editMode}
      <br />
      <h2>New Recommendation</h2>
      <label for="newName">Name</label>
      <input id="newName" type="text" bind:value={newName} />
      <label for="newLink">Link</label>
      <input id="newLink" type="text" bind:value={newLink} />
      <button on:click={addRecommendation}>Add Recommendation</button>
      <br />
      <button on:click={exportJSON}>Export Recommendations</button>
    {/if}
    {#each Object.entries(media[mediaType].sliders) as _slider}
      {#key mediaType + _slider[0]}
        <h2>{_slider[0].charAt(0).toUpperCase() + _slider[0].slice(1)}</h2>
        {#each _slider[1] as slider}
          <MoodSlider
            {slider}
            defaultValue={getDefaultValue(_slider[0], slider.name)}
            categoryName={_slider[0]}
            on:signalRecomputeRecommendations={updateSearchSetting}
          />
        {/each}
      {/key}
    {/each}
  </div>
  <div>
    <div id="recommendation-table-container">
      <table id="recommendation-table">
        <thead>
          <tr>
            <th style="width: 10%">Rank</th>
            <th style="width: 80%">Name</th>
            <th style="width: 10%">Score</th>
          </tr>
        </thead>
        <tbody>
          {#each paginatedMatches as match, index}
            <tr>
              <td>{pageNumber * recommendationsPerPage + index + 1}</td>
              {#if "link" in match.media && match.media.link.length > 0}
                <td
                  ><a href={match.media.link} target="_blank"
                    >{match.media.name}</a
                  ></td
                >
              {:else}
                <td>{match.media.name}</td>
              {/if}
              <td>{Math.floor(match.score * 100) / 100}</td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
    <div id="pagination-container">
      <button on:click={gotoFirstPage}>&lt;&lt;</button><button
        on:click={gotoPreviousPage}>&lt;</button
      >Page {pageNumber}/{lastPage}<button on:click={gotoNextPage}>&gt;</button
      ><button on:click={gotoLastPage}>&gt;&gt;</button>
    </div>
  </div>
</main>

<style>
  :root {
    font-family: Arial, Helvetica, sans-serif;
  }
  #recommendation-table-container {
    width: 100%;
    height: 90%;
  }
  #slider-container {
    user-select: none;
  }
  #recommendation-table {
    width: 100%;
    table-layout: fixed;
  }
  #recommendation-table td {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  #pagination-container {
    height: 10%;
    display: grid;
    grid-template-columns: repeat(5, 20%);
    align-items: center;
    justify-items: center;
  }
  #pagination-container button {
    height: 2rem;
    width: 2rem;
  }
  #panel-container {
    display: grid;
    grid-template-columns: auto;
  }
  @media only screen and (min-width: 960px) {
    #panel-container {
      grid-template-columns: 50% 50%;
    }
  }
</style>
