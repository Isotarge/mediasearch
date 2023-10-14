<script>
  import MoodSliderCategory from "./components/MoodSliderCategory.svelte";
  import Music from "./Music.json";
  import TV from "./TV.json";

  let mediaType = "TV";
  const media = {
    Music,
    TV,
  };
  let pageNumber = 0;
  const recommendationsPerPage = 10;
  let matches = [];
  const currentSearchSettings = {};

  $: sliders = media[mediaType].sliders;
  $: recommendations = media[mediaType].recommendations;

  // Pagination
  $: lastPage = Math.floor(recommendations.length / recommendationsPerPage);
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

    if (!(categoryName in currentSearchSettings)) {
      currentSearchSettings[categoryName] = {};
    }
    currentSearchSettings[categoryName][name] = {
      value,
      ignore,
    };
    searchMedia();
  }

  function isEnabled(categoryName, sliderName) {
    return (
      !currentSearchSettings?.[categoryName]?.[sliderName]?.ignore ?? false
    );
  }

  function checkSingleStat(media, categoryName, sliderName) {
    if (categoryName in media && sliderName in media[categoryName]) {
      const difference =
        currentSearchSettings[categoryName][sliderName].value -
        media[categoryName][sliderName];
      return Math.abs(difference);
    }
    // Note: Tweaking this value will probably be an ongoing effort
    return 10;
  }

  function computeMediaScore(media) {
    let totalDifference = 0;
    let numEnabledSliders = 0;
    for (const categoryName in currentSearchSettings) {
      for (const sliderName in currentSearchSettings[categoryName]) {
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
    matches = recommendations
      .map((media) => {
        return {
          media: media,
          score: computeMediaScore(media),
        };
      })
      .sort((a, b) => a.score - b.score);
  }

  // Editing
  let editMode = false;
  let newName = "";
  let newLink = "";

  function addRecommendation() {
    const newRecommendation = { name: newName };
    if (newLink.length > 0) {
      newRecommendation.link = newLink;
    }
    for (const categoryName in currentSearchSettings) {
      for (const sliderName in currentSearchSettings[categoryName]) {
        if (isEnabled(categoryName, sliderName)) {
          if (!(categoryName in newRecommendation)) {
            newRecommendation[categoryName] = {};
          }
          newRecommendation[categoryName][sliderName] =
            currentSearchSettings[categoryName][sliderName].value;
        }
      }
    }
    console.log(newRecommendation);
    recommendations.push(newRecommendation);
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
    <br />It was written by Isotarge as an exercise in learning
    <strike>Vue</strike> Svelte. Hope you enjoy!
  </p>
</header>
<main id="panel-container">
  <div>
    <select bind:value={mediaType}>
      {#each Object.entries(media) as media}
        <option>{media[0]}</option>
      {/each}
    </select>
    <label for="editMode">Edit Mode</label>
    <input name="editMode" type="checkbox" bind:checked={editMode} />
    {#if editMode}
      <br />
      <h2>New Recommendation</h2>
      <label for="newName">Name</label>
      <input name="newName" type="text" bind:value={newName} />
      <label for="newLink">Link</label>
      <input name="newLink" type="text" bind:value={newLink} />
      <button on:click={addRecommendation}>Add Recommendation</button>
      <br />
      <button on:click={exportJSON}>Export Recommendations</button>
    {/if}
    {#each Object.entries(sliders) as slider}
      <MoodSliderCategory
        categoryName={slider[0]}
        sliders={slider[1]}
        on:signalRecomputeRecommendations={updateSearchSetting}
      />
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
