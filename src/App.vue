<template>
  <div class="container mt-4">
    <h1 class="text-center mb-4">Google Sheets Flashcards</h1>
    <hr>

    <!-- toogle tool -->
    <div v-if="groups.length > 0" class="mb-4 text-center toggle-container">
      <template v-if="!flashcardMode">
        <div class="form-check form-check-inline flag-container">
          <input
            type="radio"
            value="japanese"
            v-model="flashcardModeSetting"
            class="form-check-input"
          />
          <label for="showJapanese" class="form-check-label flag">🇯🇵</label>
          <!-- <label class="form-check-label" v-html="japaneseEmoji"></label> -->
        </div>
        <div class="form-check form-check-inline flag-container">
          <input
            type="radio"
            value="english"
            v-model="flashcardModeSetting"
            class="form-check-input"
          />
          <label for="showEnglish" class="form-check-label flag">🇬🇧</label>
        </div>
        |
        <div class="form-check form-check-inline">
          <input
            type="radio"
            :value="true"
            v-model="displayingAll"
            class="form-check-input"
          />
          <label for="showJapanese" class="form-check-label">All</label>
        </div>
        <div class="form-check form-check-inline">
          <input
            type="radio"
            :value="false"
            v-model="displayingAll"
            class="form-check-input"
          />
          <label for="showEnglish" class="form-check-label">Marked only</label>
        </div>
      </template>
      
      
    </div>

    <!-- group looping -->
    <div v-if="groups.length > 0 && !flashcardMode" class="row gx-2 gy-2">
      <div
        v-for="(group, index) in groups"
        :key="index"
        class="col-6"
      >
        <div :style="{ backgroundColor: group.name === 'unchecked' ? 'LightSlateGray' : '' }" class="card shadow" @click="startFlashcards(group)" style="cursor: pointer;">
          <div class="card-body">
            <h5 class="card-title text-center">{{ group.name }}</h5>
            <hr>
            <section class="flex-container" style="width: 80%; margin: auto;">
              <small><i class="fa-solid fa-list"></i> <strong>{{ group.words.length }}</strong></small>
              <small v-if="group.name !== 'unchecked'"><i class="fa-solid fa-bookmark"></i> <strong>{{ group.words.filter(word => word.marked)?.length }}</strong></small>
              <small v-if="group.name !== 'unchecked'"><i class="fa-regular fa-clock"></i> <strong>{{ group.counter }}</strong></small>
            </section>
          </div>
        </div>
      </div>
    </div>


    <div v-if="groups?.length == 0" class="text-center">
      <p class="text-muted">No data loaded yet. Click the button to fetch data!</p>
    </div>

    <!-- Flashcard View -->
    <div v-if="flashcardMode" class="flashcard text-center p-4 shadow bg-light rounded">
      <div class="flex-container">
        <span class="badge bg-warning text-dark p-2">
          <i class="fa-solid fa-flag"></i> x {{ currentWord.counter }}
        </span>
        <h3 class="mb-4 text-primary">
          {{ currentGroup.name }}: {{currentWordIndex+1}}/ {{ shuffledWords.length }} 
        </h3>
        <button class="ms-3 btn btn-danger" @click="exitFlashcards"><i class="fas fa-sign-out"></i></button>
      </div>

      <!-- main word  -->
      <div>
        <div class="display-6 mb-6">
          {{ flashcardModeSetting === 'english' ? currentWord.jp : (showTranslation ? currentWord.jp : '???') }}
        </div>
        <div class="h5 mb-4">
          {{ flashcardModeSetting === 'english' ? (showTranslation ? currentWord.en : '???') : currentWord.en }}
        </div>
        <div class="h6 mb-4 text-secondary" v-if="currentWord.example">
          {{currentWord.example}}
        </div>
      </div>

      <!-- buttons  -->
      <div>
        <button class="btn btn-primary me-2" @click="backWord" :disabled="currentWordIndex === 0">
          <i class="fa-solid fa-left-long"></i>
        </button>

        <button 
          class="btn btn-secondary me-2" 
          @click="showTranslation = !showTranslation"
        >
          <i :class="showTranslation ? 'fa-solid fa-eye-slash' : 'fa-solid fa-eye'"></i>
        </button>

        <button 
          class="btn btn-warning me-2" 
          @click="toggleFlag(currentWord)"
        >
          <i :class="currentWord.flag ? 'fa-solid fa-flag' : 'fa-regular fa-flag'"></i>
        </button>

        <button 
          class="btn btn-warning me-2" 
          @click="toggleMark(currentWord)"
        >
          <i :class="currentWord.marked ? 'fa-solid fa-bookmark' : 'fa-regular fa-bookmark'"></i>
        </button>

        <button v-if="currentWordIndex !== shuffledWords.length -1" class="btn btn-success me-2" @click="nextWord">
          <i class="fa-solid fa-right-long"></i>
        </button>
       <button v-else class="btn btn-danger me-2" @click="finishRound">Finish</button>
        
      </div>
    </div>

  </div>

</template>



<script>
import twemoji from "twemoji";

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
      displayingAll: true,

      targetCell: "A555", // Default target cell
      cellValue: "Value from Vue", // Default value
      responseMessage: "", // To store the API response

      isFirstRound: false,

      // https://script.google.com/macros/s/AKfycbybFZFfo5EGapKy7svTnx5_uQ6cov5ZlfEY5oV_qauaoFJ4lXJ5JdxBih_4jclJK-CWBQ/exec
    };
  },
  computed: {
    currentWord() {
        if (!this.shuffledWords || this.shuffledWords.length === 0) {
            return { jp: "No words", en: "No words" };
        }
        return this.shuffledWords[this.currentWordIndex];
    },
    filteredGroup() {
      // Create a new array of groups where words are filtered to include only marked ones
      return this.groups.map(group => {
        return {
          ...group,
          words: group.words.filter(word => word.marked) // Include only marked words
        };
      }).filter(group => group.words.length > 0); // Remove groups with no marked words
    },
    japaneseEmoji() {
      return twemoji.parse("🇯🇵"); // Japan flag emoji
    },
    englishEmoji() {
      return twemoji.parse("🇺🇸"); // USA flag emoji
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
        let unchecked = { name: "unchecked", words: [] }; // Group for rows with row[5] == 1

        // Helper function to process a word and add it to the appropriate group
        const addWordToGroup = (row, targetGroup) => {
            if (row[0] && row[1]) {
                targetGroup.words.push({
                    jp: row[0],
                    en: row[1],
                    marked: row[4] == 1,
                    counter: row[6] || 0,
                    example: row[7],
                    
                });
            }
        };

        for (let row of values) {
            if (row[5] == 1) {
                addWordToGroup(row, unchecked);
                continue;
            }else if (row[0] && row[1] == 'groupName') {
                if (group.name) groups.push(group);
                group = { name: row[0], words: [], counter: row[6] || 0 };
                continue;
            } if (row[0] && row[1]) {
                addWordToGroup(row, group);
            }
        }

        if (group.name) groups.push(group);
        if (unchecked.words.length > 0) groups.push(unchecked); // Add the "unchecked" group if it has any words

        this.groups = groups;
    },
    shuffleArray(array) {
      // const shuffled = [...array];
      const shuffled = array.map(item => ({ ...item, flag: false }));

      for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
      }
      return shuffled;
    },

    startFlashcards(group) {
      this.isFirstRound = true
      // Shuffle the group's words and store them in shuffledWords
      // this.shuffledWords = this.shuffleArray(group.words);

      this.shuffledWords = this.shuffleArray(
        this.displayingAll ? group.words : group.words.filter(word => word.marked)
      );

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
    backWord() {
      if (this.shuffledWords.length > 0) {
        this.currentWordIndex =
          (this.currentWordIndex - 1) % this.shuffledWords.length;
        this.showTranslation = false;
      }
    },

    finishRound() {
      if(this.isFirstRound && this.currentGroup.name !== 'unchecked'){
        this.currentGroup.counter++
        this.incrementWord({jp: this.currentGroup.name, en: 'groupName'})
      }
      this.isFirstRound = false
      // Filter flagged words from shuffledWords
      const flaggedWords = this.shuffledWords.filter(word => word.flag);

      if (flaggedWords.length === 0) return this.exitFlashcards();

      // Shuffle the flagged words and reset state
      this.shuffledWords = this.shuffleArray(flaggedWords);
      this.currentWordIndex = 0;
      this.showTranslation = false;
    },

    toggleMark(word) {
      word.marked = !word.marked;
      if (word.marked) this.toggleFlag(word)

      // Build the API URL with the action parameter for toggleMark
      const baseUrl =
        "https://script.google.com/macros/s/AKfycbyBcvDCo3fYslOGZeJLoDlL8TayF1YwBAyNPh-xpLp7YsBwF3dvlOUCjxSVO7hiFfTBSA/exec";
      const url = `${baseUrl}?callback=jsonpCallback&action=toggleMark&english=${encodeURIComponent(
        word.en
      )}&japanese=${encodeURIComponent(word.jp)}`;

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (toggleMark):", data);
      };

      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };
    },

    toggleFlag(word){
      word.flag = !word.flag
      if(!word.flag) return
      this.incrementWord(word)
    },

    incrementWord(word) {
      console.log(word);
      word.counter++
      // Build the API URL with the action parameter for increment
      const baseUrl =
        "https://script.google.com/macros/s/AKfycbyBcvDCo3fYslOGZeJLoDlL8TayF1YwBAyNPh-xpLp7YsBwF3dvlOUCjxSVO7hiFfTBSA/exec";
      const url = `${baseUrl}?callback=jsonpCallback&action=increment&english=${encodeURIComponent(word.en)}&japanese=${encodeURIComponent(word.jp)}`;

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (increment):", data);
      };

      // Dynamically add a <script> tag to call the JSONP API
      const script = document.createElement("script");
      script.src = url; // Set the API URL
      script.async = true; // Load asynchronously
      document.body.appendChild(script);

      // Clean up the <script> tag after the request
      script.onload = () => {
        document.body.removeChild(script); // Remove the script tag
      };
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
    console.clear()
    this.fetchData();
  },
  watch: {
    flashcardMode(newVal) {
      const htmlBodyStyles = document.documentElement.style;

      if (newVal) {
        htmlBodyStyles.height = "100%";
        htmlBodyStyles.margin = "0";
        htmlBodyStyles.overflow = "hidden";
      } else {
        htmlBodyStyles.height = "";
        htmlBodyStyles.margin = "";
        htmlBodyStyles.overflow = "";
      }
    },
  }
};


</script>

<style>
#app {
  height: 100vh; /* Ensures the app takes the full viewport height */
  display: flex; /* Optional: For centering or flex layouts */
  flex-direction: column; /* Optional: Ensures children stack vertically */
}

.toggle-container{
  display: flex;
  justify-content: space-around !important;
  align-items: center;
}

.flag-container{
  display: flex !important;
  align-items: center;
  gap: 5px;
  
}

.form-check-inline{
  margin-right: unset !important;
}

.flag{
  font-size: 2em;
}

.flex-container{
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin-bottom: 10px;
}

.text-center{
  justify-content: center;
}

.flex-container span{
  font-size: 1em;
}

.flex-container h3{
  margin-bottom: unset !important;
}

.card-title{
  margin-bottom: unset !important;
}

.card-body{
  padding: 10px !important;
}

.card-body hr{
  margin: .5em 0;
}

.flashcard{
  border: 1px solid lightslategray;
}
</style>