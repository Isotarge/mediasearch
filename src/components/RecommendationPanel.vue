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
          <tbody>
          <tr v-for='(match, index) in matches' :key='index'>
          <td>{{ index + 1 }}</td>
          <template v-if='Object.prototype.hasOwnProperty.call(match[0], "link")'>
            <td><a :href="match[0].link" target='_blank'>{{match[0].name}}</a></td>
          </template>
          <template v-else>
            <td>{{ match[0].name }}</td>
          </template>
          <td>{{ Math.floor(match[1] * 100) / 100 }}</td>
          </tr>
          </tbody>
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
  data: function() {
    return {
      numRecommendations: 10,
      matches: [],
      exportedSearchSettings: '',
    };
  },
  props: {
    MusicRecommendations: Array,
    currentSearchSettings: {
      default() {
        return {};
      },
      type: Object
    },
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
        name: "",
        link: "",
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
      this.matches = [];
      for (let i = 0; i < Math.min(this.numRecommendations, matches.length); i += 1) {
        this.matches.push(matches[i]);
      }
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
