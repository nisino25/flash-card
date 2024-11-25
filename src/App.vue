<template>
  <div>
    <h1>Google Sheets Data</h1>
    <button @click="fetchData">Fetch Data</button>
    <div v-if="groups.length > 0">
      <div v-for="(group, index) in groups" :key="index" class="group">
        <h2>{{ group.name }}</h2>
        <ul>
          <li v-for="(word, wordIndex) in group.words" :key="wordIndex">
            {{ word.jp }} - {{ word.en }}
          </li>
        </ul>
      </div>
    </div>
    <div v-else>
      <p>No data loaded yet. Click the button to fetch data!</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      groups: [], // Stores processed group data
      apiKey: "AIzaSyC1jxmY8gKcMDMbgyYBb1rd72Jq_QHVhYg", // Replace with your API Key
      sheetId: "1imzTupVh0DnKkKpeBXzZgJFC6PIZIE83YFLSUqXp0YE", // Replace with your Sheet ID
    };
  },
  methods: {
    async fetchData() {
      const url = `https://sheets.googleapis.com/v4/spreadsheets/${this.sheetId}/values/Sheet1?key=${this.apiKey}`;
      try {
        const response = await fetch(url);
        const data = await response.json();

        if (data.values) {
          this.processData(data.values);
        } else {
          console.error("No data found!");
        }
      } catch (error) {
        console.error("Error fetching data:", error);
      }
    },
    processData(values) {
      let groups = [];
      let group = { name: "", words: [] };

      for (let row of values) {
        if (row[0] && row[1] && row[2]) {
          // If all columns have data, it's a word entry
          group.words.push({ jp: row[0], en: row[1] });
        } else if (row[0] && !row[1] && !row[2]) {
          // If only the first column has data, it's a new group
          if (group.name) groups.push(group); // Save previous group
          group = { name: row[0], words: [] }; // Start new group
        }
      }

      if (group.name) groups.push(group); // Add last group
      this.groups = groups;

      console.log("Processed Groups:", this.groups);
    },
  },
};
</script>

<style>
.group {
  margin-bottom: 20px;
}
</style>
