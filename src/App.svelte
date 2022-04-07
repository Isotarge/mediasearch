<script>
  import MoodSliderCategory from "./components/MoodSliderCategory.svelte";
  import Music from "./Music.json";
  import TV from "./TV.json";

  let mediaType = "Music";
  const media = {
    Music,
    TV,
  };

  $: sliders = media[mediaType].sliders;
  $: recommendations = media[mediaType].recommendations;

  let numRecommendations = 10;
  let matches = [];
  const currentSearchSettings = {};
  let exportedSearchSettings = "";

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
    updateExportedSearchTextarea();
    searchMedia();
  }

  function isEnabled(categoryName, sliderName) {
    return (
      !currentSearchSettings?.[categoryName]?.[sliderName]?.ignore ?? false
    );
  }

  function updateExportedSearchTextarea() {
    const exportedSearch = {
      name: "",
      link: "",
    };
    for (const categoryName in currentSearchSettings) {
      const searchSetting = currentSearchSettings[categoryName];
      for (const sliderName in searchSetting) {
        if (isEnabled(categoryName, sliderName)) {
          const exportedSlider = searchSetting[sliderName];
          if (!(categoryName in exportedSearch)) {
            exportedSearch[categoryName] = {};
          }
          exportedSearch[categoryName][sliderName] = exportedSlider.value;
        }
      }
    }
    exportedSearchSettings = JSON.stringify(exportedSearch, null, "\t");
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
    let score = 0;
    let numScores = 0;
    for (const categoryName in currentSearchSettings) {
      for (const sliderName in currentSearchSettings[categoryName]) {
        if (isEnabled(categoryName, sliderName)) {
          numScores++;
          score += checkSingleStat(media, categoryName, sliderName);
        }
      }
    }
    if (numScores > 0) {
      // Average difference
      return score / numScores;
    }
    return 0;
  }

  function searchMedia() {
    // Compute the new matches
    const _matches = recommendations
      .map((media) => {
        return {
          media: media,
          score: computeMediaScore(media),
        };
      })
      .sort((a, b) => a.score - b.score);

    // Clamp number of matches to numRecommendations
    matches = _matches.slice(0, Math.min(numRecommendations, _matches.length));
  }
</script>

<header>
  <h1>Media Search</h1>
  <p>
    This tool will take your current feelings, desires, style preferences, and
    spit out some media it thinks you'll like.
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
    {#each Object.entries(sliders) as slider}
      <MoodSliderCategory
        categoryName={slider[0]}
        sliders={slider[1]}
        on:signalRecomputeRecommendations={updateSearchSetting}
      />
    {/each}
  </div>
  <div>
    <label for="num-recommendations">Number of recommendations</label>
    &nbsp;
    <input
      name="num-recommendations"
      type="number"
      min="1"
      step="1"
      bind:value={numRecommendations}
      on:change={searchMedia}
    />
    <div id="recommendation-container">
      <div id="recommendation-table-container">
        <table>
          <thead>
            <tr>
              <th>Rank</th>
              <th>Name</th>
              <th>Score</th>
            </tr>
          </thead>
          <tbody>
            {#each matches as match, index}
              <tr>
                <td>{index + 1}</td>
                {#if "link" in match.media}
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
      <textarea value={exportedSearchSettings} />
    </div>
  </div>
</main>

<style>
  :root {
    font-family: Arial, Helvetica, sans-serif;
  }
  textarea {
    font-family: "Courier New", Courier, monospace;
  }
  #panel-container {
    display: grid;
    grid-template-columns: 50% 50%;
  }
  #recommendation-container {
    height: 100%;
    display: grid;
    grid-template-rows: 50% 50%;
  }
  #recommendation-table-container {
    max-height: 100%;
    height: auto;
    overflow: scroll;
  }
</style>
