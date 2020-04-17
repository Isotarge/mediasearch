<template>
  <div id="recommendationPanel">
    Number of recommendations:
    <input
      type="number"
      min="1"
      step="1"
      v-model="numRecommendations"
      @change="searchMedia"
    />
    <div v-html="searchOutput">
      <!--<Recommendation/>-->
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
    searchOutput: String
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
      this.searchMedia();
    },
    checkSingleStat(media, categoryName, sliderName) {
      if (Object.prototype.hasOwnProperty.call(media, categoryName)) {
        if (
          Object.prototype.hasOwnProperty.call(media[categoryName], sliderName)
        ) {
          var difference =
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
      var score = 0;
      var scores = 0;
      for (var categoryName in this.currentSearchSettings) {
        for (var sliderName in this.currentSearchSettings[categoryName]) {
          if (this.isEnabled(categoryName, sliderName)) {
            scores++;
            score += this.checkSingleStat(media, categoryName, sliderName);
          }
        }
      }
      if (scores > 0) {
        // Average difference
        return score / scores;
      }
      return 0;
    },
    searchMedia() {
      var matches = [];
      var _this = this;
      this.MusicRecommendations.forEach(function(media) {
        var score = _this.matchMedia(media);
        matches.push([media, score]);
      });
      matches.sort(function(a, b) {
        return a[1] - b[1];
      });
      var matchString = "";
      matchString += "<table>";
      matchString += "<thead>\n";
      matchString += "	<tr>\n";
      matchString += "		<td>Rank</td>\n";
      matchString += "		<td>Name</td>\n";
      matchString += "		<td>Score</td>\n";
      matchString += "	</tr>\n";
      matchString += "</thead>\n";
      matchString += "<tbody>\n";
      var numMatches = 0;
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
      matchString += "</tbody>\n";
      matchString += "</table>";
      this.searchOutput = matchString;
    }
  }
};
</script>

<style scoped>
</style>
