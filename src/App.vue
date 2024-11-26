<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Google Sheets Flashcards</h1>

    <!-- Toggle for Translation Mode -->
    <div v-if="groups.length > 0" class="mb-4 text-center">
      <label class="form-check-label me-2">Show:</label>
      <div class="form-check form-check-inline">
        <input
          type="radio"
          id="showJapanese"
          value="japanese"
          v-model="flashcardModeSetting"
          class="form-check-input"
        />
        <label for="showJapanese" class="form-check-label">Japanese</label>
      </div>
      <div class="form-check form-check-inline">
        <input
          type="radio"
          id="showEnglish"
          value="english"
          v-model="flashcardModeSetting"
          class="form-check-input"
        />
        <label for="showEnglish" class="form-check-label">English</label>
      </div>
    </div>

    <div v-if="groups.length > 0 && !flashcardMode" class="row gx-2 gy-2">
  <div
    v-for="(group, index) in groups"
    :key="index"
    class="col-6"
  >
    <div class="card shadow" @click="startFlashcards(group)" style="cursor: pointer;">
      <div class="card-body">
        <h5 class="card-title text-center">
          {{ group.name }} <small class="text-muted">({{ group.words.length }})</small>
        </h5>
      </div>
    </div>
  </div>
</div>


    <div v-if="groups?.length == 0" class="text-center">
      <p class="text-muted">No data loaded yet. Click the button to fetch data!</p>
    </div>

    <!-- Flashcard View -->
    <div v-if="flashcardMode" class="flashcard text-center p-4 shadow bg-light rounded">
      <h2 class="mb-4 text-primary">Flashcard Mode: {{ currentGroup.name }}</h2>
      <div class="display-4 mb-4">
        {{ flashcardModeSetting === 'english' ? currentWord.jp : (showTranslation ? currentWord.jp : '???') }}
      </div>
      <div class="h5 mb-4">
        {{ flashcardModeSetting === 'english' ? (showTranslation ? currentWord.en : '???') : currentWord.en }}
      </div>
      <div>
        <button class="btn btn-success me-2" @click="nextWord">Next</button>
        <button class="btn btn-secondary me-2" @click="toggleTranslation">Toggle Visibility</button>
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
      apiKey: process.env.VUE_APP_API_KEY,
      sheetId: process.env.VUE_APP_SHEET_ID,
      flashcardMode: false, // Toggles flashcard mode
      currentGroup: null, // Current group in flashcard mode
      currentWordIndex: 0, // Index of the current word
      showTranslation: false, // Whether to show the translation
      flashcardModeSetting: "english", // Default to hiding Japanese
      shuffledWords: [],
    };
  },
  computed: {
    currentWord() {
      if (!this.shuffledWords || this.shuffledWords.length === 0) {
        return { jp: "No words", en: "No words" };
      }
      return this.shuffledWords[this.currentWordIndex];
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
          group.words.push({ jp: row[0], en: row[1] });
        } else if (row[0] && !row[1]) {
          if (group.name) groups.push(group);
          group = { name: row[0], words: [] };
        }
      }

      if (group.name) groups.push(group);
      this.groups = groups;
    },
    shuffleArray(array) {
      const shuffled = [...array];
      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }
      return shuffled;
    },

    startFlashcards(group) {
      // Shuffle the group's words and store them in shuffledWords
      this.shuffledWords = this.shuffleArray(group.words);

      this.flashcardMode = true;
      this.currentGroup = group;
      this.currentWordIndex = 0;
      this.showTranslation = false;
    },

    nextWord() {
      if (this.shuffledWords.length > 0) {
        this.currentWordIndex =
          (this.currentWordIndex + 1) % this.shuffledWords.length;
        this.showTranslation = false;
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
      this.shuffledWords = [];
    },
  },
  mounted() {
    this.fetchData();
  },

};


</script>

<style>
html, body {
  height: 100%; /* Ensures html and body take the full height */
  margin: 0; /* Removes default margin */
  overflow: hidden; /* Prevents scrolling or overflow */
}

#app {
  height: 100vh; /* Ensures the app takes the full viewport height */
  display: flex; /* Optional: For centering or flex layouts */
  flex-direction: column; /* Optional: Ensures children stack vertically */
}
</style>