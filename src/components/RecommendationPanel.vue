<template>
  <div id="recommendationPanel">
    <label>Number of recommendations</label>
    &nbsp;
    <input
      type="number"
      min="1"
      step="1"
      v-model="numRecommendations"
      @change="searchMedia"
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
          <tbody v-html="searchOutput"></tbody>
        </table>
      </div>
      <textarea v-model="exportedSearchSettings"></textarea>
    </div>
  </div>
</template>

<script>
//import Recommendation from './Recommendation.vue'

export default {
  name: "RecommendationPanel",
  components: {
    //Recommendation
  },
  props: {
    numRecommendations: Number,
    MusicRecommendations: Array,
    currentSearchSettings: {
      default() {
        return {};
      },
      type: Object
    },
    searchOutput: String,
    exportedSearchSettings: String
  },
  methods: {
    updateSearchSetting(categoryName, sliderName, sliderValue, ignore) {
      if (
        !Object.prototype.hasOwnProperty.call(
          this.currentSearchSettings,
          categoryName
        )
      ) {
        this.currentSearchSettings[categoryName] = {};
      }
      this.currentSearchSettings[categoryName][sliderName] = {
        value: sliderValue,
        ignore: ignore
      };
      let exportedSearch = {
        name: ""
      };
      for (let exportedCategoryName in this.currentSearchSettings) {
        if (exportedCategoryName === "name") {
          continue;
        }
        const searchSetting = this.currentSearchSettings[exportedCategoryName];
        for (let exportedSliderName in searchSetting) {
          const exportedSlider = searchSetting[exportedSliderName];
          if (
            Object.hasOwnProperty.call(exportedSlider, "ignore") &&
            Object.hasOwnProperty.call(exportedSlider, "value")
          ) {
            if (!exportedSlider.ignore) {
              if (
                !Object.hasOwnProperty.call(
                  exportedSearch,
                  exportedCategoryName
                )
              ) {
                exportedSearch[exportedCategoryName] = {};
              }
              exportedSearch[exportedCategoryName][exportedSliderName] =
                exportedSlider.value;
            }
          }
        }
      }
      this.exportedSearchSettings = JSON.stringify(exportedSearch, null, "\t");
      this.searchMedia();
    },
    checkSingleStat(media, categoryName, sliderName) {
      if (Object.prototype.hasOwnProperty.call(media, categoryName)) {
        if (
          Object.prototype.hasOwnProperty.call(media[categoryName], sliderName)
        ) {
          let difference =
            this.currentSearchSettings[categoryName][sliderName].value -
            media[categoryName][sliderName];
          difference = Math.abs(difference);
          return difference;
        }
      }
      // Note: Tweaking this value will probably be an ongoing effort
      return 10;
    },
    isEnabled(categoryName, sliderName) {
      if (
        Object.prototype.hasOwnProperty.call(
          this.currentSearchSettings,
          categoryName
        )
      ) {
        if (
          Object.prototype.hasOwnProperty.call(
            this.currentSearchSettings[categoryName],
            sliderName
          )
        ) {
          if (
            Object.prototype.hasOwnProperty.call(
              this.currentSearchSettings[categoryName][sliderName],
              "ignore"
            )
          ) {
            return !this.currentSearchSettings[categoryName][sliderName].ignore;
          }
        }
      }
      return false;
    },
    matchMedia(media) {
      let score = 0;
      let numScores = 0;
      for (let categoryName in this.currentSearchSettings) {
        for (let sliderName in this.currentSearchSettings[categoryName]) {
          if (this.isEnabled(categoryName, sliderName)) {
            numScores++;
            score += this.checkSingleStat(media, categoryName, sliderName);
          }
        }
      }
      if (numScores > 0) {
        // Average difference
        return score / numScores;
      }
      return 0;
    },
    searchMedia() {
      let matches = [];
      let _this = this;
      this.MusicRecommendations.forEach(function(media) {
        const score = _this.matchMedia(media);
        matches.push([media, score]);
      });
      matches.sort(function(a, b) {
        return a[1] - b[1];
      });
      let matchString = "";

      let numMatches = 0;
      matches.forEach(function(match) {
        if (numMatches < _this.numRecommendations) {
          numMatches++;
          matchString += "<tr>\n";
          matchString += "	<td>" + numMatches + "</td>\n";
          if (Object.prototype.hasOwnProperty.call(match[0], "link")) {
            matchString +=
              "	<td><a href='" +
              match[0].link +
              "' target='_blank'>" +
              match[0].name +
              "</a></td>\n";
          } else {
            matchString += "	<td>" + match[0].name + "</td>\n";
          }
          matchString += "	<td>" + Math.floor(match[1] * 100) / 100 + "</td>\n";
          matchString += "</tr>\n";
        }
      });
      this.searchOutput = matchString;
    }
  }
};
</script>

<style scoped>
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
