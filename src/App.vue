<script setup>
import words from "./words.js";
import { ref } from "vue";

let randomWord = ref([]);
let wordBank = [];
let unitsPracticedToday = 0;
let unitsPracticedYesterday = 0;
// see if wordBank is in localStorage, if so, load it,  if not, set it to the imported words
if (!localStorage.getItem("wordBank")) {
  wordBank.value = words;
} else {
  // if it is in localStorage, set the wordBank to the localStorage value
  wordBank.value = JSON.parse(localStorage.getItem("wordBank"));
}

// same with localStorage stats
let stats = {};
if (!localStorage.getItem("stats")) {
  stats = {
    counter: 0,
  };
} else {
  stats = JSON.parse(localStorage.getItem("stats"));
}

function getRandomWord() {
  isRevealed.value = false;
  console.log("picking word from", wordBank);
  const newWord =
    wordBank.value[Math.floor(Math.random() * wordBank.value.length)];
  // if newWord is missing property evaluationType, randomly give it 'anki' or 'likert'
  if (!newWord.evaluationType) {
    newWord.evaluationType = "anki";
  }
  valueEase.value = null;
  valueCorrect.value = null;
  valueAnki.value = null;

  randomWord.value = newWord;
  // save the wordBank to localStorage (doesn't make that much sense here in the code but whatever)
  localStorage.setItem("wordBank", JSON.stringify(wordBank.value));
}

let isRevealed = ref(false);

let valueEase = ref(null);
let valueCorrect = ref(null);
let valueAnki = ref(null);

getRandomWord();

function evaluateScore() {
  if (randomWord.value.evaluationType == "anki") {
    if (valueAnki.value == null) return;
    // add a log to the word's repetition property - if the property doesn't exist, create it
    if (!randomWord.value.repetitions) {
      randomWord.value.repetitions = [];
    }
    randomWord.value.repetitions.push({
      date: new Date().toISOString(),
      score: valueAnki.value,
    });
    stats.counter++;
    localStorage.setItem("stats", JSON.stringify(stats));
    getRandomWord();
  } else {
    if (valueEase.value == null || valueCorrect.value == null) return;
    // add a log to the word's repetition property - if the property doesn't exist, create it
    if (!randomWord.value.repetitions) {
      randomWord.value.repetitions = [];
    }
    randomWord.value.repetitions.push({
      date: new Date().toISOString(),
      ease: valueEase.value,
      correct: valueCorrect.value,
    });
    stats.counter++;
    localStorage.setItem("stats", JSON.stringify(stats));
    getRandomWord();
  }
}
</script>

<template>
  <small> Practiced {{ stats.counter }} times so far </small>
  <div
    class="card bg-gray-600 shadow-xl m-4 flex flex-col items-center w-full max-w-screen-xl"
  >
    <div class="card-body">
      <h2 class="card-title my-2 text-6xl">{{ randomWord.q }}</h2>
      <div :class="{ hidden: !isRevealed }" class="flex items-center gap-2">
        <p class="my-2 text-xl">
          {{ randomWord.a }}
        </p>
   
      </div>
      <div class="card-actions justify-center mt-6 pt-2">
        <button
          class="btn btn-primary"
          @click="isRevealed = true"
          v-if="!isRevealed"
        >
          Reveal
        </button>
        <div class="" v-else>
          <div
            class="flex gap-2 justify-center items-center"
            v-if="randomWord.evaluationType == 'anki'"
          >
            <button
              class="btn"
              @click="
                valueAnki = 0;
                evaluateScore();
              "
              :class="{ 'btn-primary': valueAnki == 0 }"
            >
              Again
            </button>
            <button
              class="btn"
              @click="
                valueAnki = 1;
                evaluateScore();
              "
              :class="{ 'btn-primary': valueAnki == 1 }"
            >
              Hard
            </button>
            <button
              class="btn"
              @click="
                valueAnki = 2;
                evaluateScore();
              "
              :class="{ 'btn-primary': valueAnki == 2 }"
            >
              Good
            </button>
            <button
              class="btn"
              @click="
                valueAnki = 3;
                evaluateScore();
              "
              :class="{ 'btn-primary': valueAnki == 3 }"
            >
              Easy
            </button>
          </div>
      
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
