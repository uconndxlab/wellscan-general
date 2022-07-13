<template>
  <div id="app">
    <label for="primarySearch">
      Search for a Food:
      <v-select :options="foodOptions" v-model="selectedFood" placeholder="Enter food here..."></v-select>
    </label>

    <div class="results" v-if="selectedFood">
      <p>Category: {{ results[selectedFood].category }}</p>
      <p>Rank: {{ results[selectedFood].rank }}</p>
    </div>
  </div>
</template>

<script>
import { get } from 'axios'
import 'vue-select/dist/vue-select.css';

export default {
  name: 'home',
  data() {
    return {
      gapiKey: process.env.VUE_APP_GOOGLE_SHEETS_API_KEY,
      spreadsheetId: process.env.VUE_APP_SPREADSHEET_ID,
      foodOptionsRaw: [],
      results: {},
      selectedFood: ''
    }
  },
  async created() {

    // This one can get general information on the spreadsheet, such as column and row ranges.
    // const g = await get(`https://sheets.googleapis.com/v4/spreadsheets/${this.spreadsheetId}?key=${this.gapiKey}`)

    // Retrieves values in the range provided.
    const g = await get(`https://sheets.googleapis.com/v4/spreadsheets/${this.spreadsheetId}/values/A1:E1000?key=${this.gapiKey}`)

    // Loop our values, transform readable/usable data.
    for (let index = 0; index < g.data.values.length; index++) {

      // Retrieve one single row.
      const element = g.data.values[index];
      
      // Retrive the name of the food
      const foodName = element[1].toLowerCase()

      // Add to our list of selectable foods.
      this.foodOptionsRaw.push(foodName)

      // Save a result object with the food as key
      this.results[foodName] = {
        category: (element[2] === undefined) ? 'N/A' : element[2],
        rank: (element[3] === undefined) ? 'Unranked' : element[3],
      }
    }
  },
  computed: {
    foodOptions() {
      return this.foodOptionsRaw
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  max-width: 800px;
  margin-left: auto;
  margin-right: auto;
}
</style>
