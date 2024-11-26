<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Google Sheets Flashcards</h1>
    <!-- <div class="text-center mb-4">
      <button class="btn btn-primary" @click="fetchData">Fetch Data</button>
    </div> -->

    <div v-if="groups.length > 0" class="row">
      <div v-for="(group, index) in groups" :key="index" class="col-6 mb-4">
        <div class="card shadow" @click="startFlashcards(group)" style="cursor: pointer;">
          <div class="card-body">
            <h5 class="card-title text-center">
              {{ group.name }} <small class="text-muted">({{ group.words.length }})</small>
            </h5>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="text-center">
      <p class="text-muted">No data loaded yet. Click the button to fetch data!</p>
    </div>


    <!-- Flashcard View -->
    <div v-if="flashcardMode" class="flashcard text-center p-4 shadow bg-light rounded">
      <h2 class="mb-4 text-primary">Flashcard Mode: {{ currentGroup.name }}</h2>
      <div class="display-4 mb-4">
        {{ currentWord.jp }}
      </div>
      <div class="h5 mb-4">
        {{ showTranslation ? currentWord.en : "???" }}
      </div>
      <div>
        <button class="btn btn-success me-2" @click="nextWord">Next</button>
        <button class="btn btn-secondary me-2" @click="toggleTranslation">Toggle Translation</button>
        <button class="btn btn-danger" @click="exitFlashcards">Exit</button>
      </div>
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
      flashcardMode: false, // Toggles flashcard mode
      currentGroup: null, // Current group in flashcard mode
      currentWordIndex: 0, // Index of the current word
      showTranslation: false, // Whether to show the translation
    };
  },
  computed: {
    currentWord() {
      if (!this.currentGroup || this.currentGroup.words.length === 0) {
        return { jp: "No words", en: "No words" };
      }
      return this.currentGroup.words[this.currentWordIndex];
    },
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
        if (row[0] && row[1]) {
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
    startFlashcards(group) {
      this.flashcardMode = true;
      this.currentGroup = group;
      this.currentWordIndex = 0;
      this.showTranslation = false;
    },
    nextWord() {
      console.log('here');
      if (this.currentGroup && this.currentGroup.words.length > 0) {
        this.currentWordIndex =
          (this.currentWordIndex + 1) % this.currentGroup.words.length;
        this.showTranslation = false; // Hide translation for the next word
      }
    },
    toggleTranslation() {
      this.showTranslation = !this.showTranslation;
    },
    exitFlashcards() {
      this.flashcardMode = false;
      this.currentGroup = null;
      this.currentWordIndex = 0;
      this.showTranslation = false;
    },
  },
  mounted(){
    console.clear()
    this.fetchData();
  }
};

</script>

<style>

</style>
