<script setup>
import words from "./words.js";
import { ref } from "vue";

let randomWord = ref([]);
let wordBank = ref([]);
let unitsPracticedToday = 0;
let unitsPracticedYesterday = 0;
// see if wordBank is in localStorage, if so, load it,  if not, set it to the imported words
  wordBank.value = words;

// if (!localStorage.getItem("wordBank")) {
//   wordBank.value = words;
// } else {
//   // if it is in localStorage, set the wordBank to the localStorage value
//   wordBank.value = JSON.parse(localStorage.getItem("wordBank"));
// }

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
  const dueWords = wordBank.value.filter((word) => {
    return word.dueDate < new Date().getTime() / 1000 || !word.dueDate;
  });
  console.log("dueWords:", dueWords.length);
  console.log("dueWords", dueWords);
  isRevealed.value = false;
  console.log("picking word from", wordBank);
  if (dueWords.length == 0) {
    randomWord.value = {};
    return;
  }
  const newWord = dueWords[Math.floor(Math.random() * dueWords.length)];
  // if newWord is missing property evaluationType, randomly give it 'anki' or 'likert'
  if (!newWord.evaluationType) {
    newWord.evaluationType = "anki";
    newWord.streak = 0;
    newWord.interval = 10;
    newWord.dueDate = 0;
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
  // if ankiValue is 0, half interval (minimum 10), if ankiValue is 2, double interval
  if (valueAnki.value == 0) {
    randomWord.value.interval = Math.max(randomWord.value.interval / 2, 10);
    randomWord.value.streak = 0;
  } else if (valueAnki.value == 2) {
    randomWord.value.streak++;
    randomWord.value.interval = randomWord.value.interval * 2 * randomWord.value.streak;
  }

  randomWord.value.dueDate = new Date().getTime() / 1000 + randomWord.value.interval;
  if (valueAnki.value == null) return;
  // add a log to the word's repetition property - if the property doesn't exist, create it
  if (!randomWord.value.repetitions) {
    randomWord.value.repetitions = [];
  }
  // randomWord.value.repetitions.push({
  //   date: new Date().toISOString(),
  //   score: valueAnki.value,
  // });
  stats.counter++;
  localStorage.setItem("stats", JSON.stringify(stats));
  localStorage.setItem("wordBank", JSON.stringify(wordBank.value));
  getRandomWord();
}
</script>

<template>
  <small> Practiced {{ stats.counter }} times so far </small>
  <div class=""  v-if="!randomWord.q">nothing more to practice.</div>

  <div
    class="card bg-gray-600 shadow-xl m-4 flex flex-col items-center w-full max-w-screen-xl" v-else
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
              Wrong
            </button>
            <button
              class="btn"
              @click="
                valueAnki = 1;
                evaluateScore();
              "
              :class="{ 'btn-primary': valueAnki == 1 }"
            >
              Kind Of
            </button>
            <button
              class="btn"
              @click="
                valueAnki = 2;
                evaluateScore();
              "
              :class="{ 'btn-primary': valueAnki == 2 }"
            >
              Correct
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <br>
</template>

<style scoped></style>
