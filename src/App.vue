<script setup>
import { ref, onMounted, watch } from "vue";
import Square from "./components/Square.vue";
import Controls from "./components/Controls.vue";

let id = 0;
let gameProgression = ref(-1);
let count = ref(0);

let correctPosition = ref(0);
let incorrectPosition = ref(0);
let correctSound = ref(0);
let incorrectSound = ref(0);
let testRun = false;
const squares = ref([
  { id: id++ },
  { id: id++ },
  { id: id++ },
  { id: id++ },
  { id: id++ },
  { id: id++ },
]);

let activate = ref([]);
let nBack = ref("2");
let faktor = ref("12");
let numberOfRounds = ref(nBack.value * faktor.value);
let nBackPosition = false;
let nBackSound = false;
let roundLength = testRun ? testPositions.length : numberOfRounds.value;
const letters = ["a", "b", "c", "d", "e", "f"];
const testPositions = [0, 1, 0, 3, 4, 0, 4, 1, 1, 1];
const testLetters = ["a", "b", "c", "d", "e", "f", "e", "a", "b", "a"];
const selectedTest = ref("2");

const optionsTest = ref([
  { text: "2", value: "2" },
  { text: "3", value: "3" },
  { text: "4", value: "4" },
]);

onMounted(() => {
  //startGame();
});

if (testRun) {
  for (let i = 0; i < roundLength; i++) {
    let squareNum = testPositions[i];
    let squareLetter = testLetters[i];
    nBackPosition = nBackPositionCheck(squareNum);
    nBackSound = nBackSoundCheck(squareLetter);

    let obj = {
      square: squareNum,
      isPos: nBackPosition,
      sound: squareLetter,
      posCorrect: false,
      isSound: nBackSound,
      soundCorrect: false,
    };

    activate.value.push(obj);
  }
} else {
  createChallenge();
}

function calculateTrials(e) {
  let elementClass = e.srcElement.classList[0];
  if (elementClass == "faktorInput") {
    faktor.value = e.target.value;
  } else if (elementClass == "dualSelectInput") {
    nBack.value = e.target.value;
  }

  numberOfRounds.value = nBack.value * faktor.value;
  roundLength = numberOfRounds.value;
  createChallenge();
}

function createChallenge() {
  activate = ref([]);
  for (let i = 0; i < roundLength; i++) {
    let squareNum = Math.floor(Math.random() * 5);
    let squareLetter = Math.floor(Math.random() * 5);
    nBackPosition = nBackPositionCheck(squareNum);

    let obj = {
      square: squareNum,
      isPos: nBackPosition,
      sound: letters[squareLetter],
      posCorrect: false,
      isSound: nBackSound,
      soundCorrect: false,
    };
    activate.value.push(obj);
  }
  console.log(activate);
}

function nBackPositionCheck(squareNum) {
  if (activate.value.length > nBack.value - 1) {
    let nBackCheck = activate.value.length - nBack.value;
    if (squareNum == activate.value[nBackCheck].square) {
      return true;
    }
  }
  return false;
}

function nBackSoundCheck(sound) {
  if (activate.value.length > 1) {
    let nBackCheck = activate.value.length - nBack.value;

    if (sound == activate.value[nBackCheck].sound) {
      return true;
    }
  }
  return false;
}

function startGame(e) {
  e.target.setAttribute("disabled", true);
  gameProgression.value = -1;

  /*if (gameProgression.value == 0) {
  }*/
  let showSquare = setInterval(() => {
    if (gameProgression.value > 0) {
      let prev = activate.value[gameProgression.value];
      positionScoring(prev);
      soundScoring(prev);
    }

    if (roundLength - 1 == gameProgression.value) {
      endSession(showSquare);
      return;
    }

    gameProgression.value++;
    let currentN = activate.value[gameProgression.value];
    squareShowingHiding(currentN);
    gameSound(currentN);

    count.value++;
  }, 3000);
}

function positionCheck() {
  activate.value[gameProgression.value].posCorrect = true;
}

function soundCheck() {
  activate.value[gameProgression.value].soundCorrect = true;
}

function gameSound(currentN) {
  let speech = new SpeechSynthesisUtterance();
  var synth = window.speechSynthesis;
  let voices = window.speechSynthesis.getVoices();
  speech.voice = voices[1];
  speech.lang = "en";
  speech.rate = 0.5;

  speech.text = currentN.sound;
  synth.speak(speech);
}

function soundScoring(prev) {
  if (prev.isSound == prev.soundCorrect) {
    correctSound.value++;
  } else {
    incorrectSound.value++;
  }
}

function positionScoring(prev) {
  if (prev.isPos == prev.posCorrect) {
    correctPosition.value++;
  } else {
    incorrectPosition.value++;
  }
}

function squareShowingHiding(currentN) {
  let elemId = "#square-" + currentN.square;
  let currentActiveSquare = document.querySelector(elemId);
  currentActiveSquare.classList.add("active");

  setTimeout(() => {
    if (currentActiveSquare != null) {
      currentActiveSquare.classList.remove("active");
    }
  }, 500);
}

function endSession(showSquare) {
  document.querySelector(".finish-popup").classList.add("display-popup");
  clearInterval(showSquare);
}

function closeFinishPopup() {
  document.querySelector(".finish-popup").classList.remove("display-popup");
  document.querySelector(".playButton").removeAttribute("disabled");
  count.value = 0;
}
</script>

<template>
  <header></header>
  <main>
    <div class="main-container">
      <div class="playground">
        <p>
          {{ count }} of
          {{ numberOfRounds }}
        </p>
        <p>Faktor:</p>
        <input
          className="faktorInput"
          :value="faktor"
          @input="calculateTrials"
          type="text"
        />
        <p>dual n back:</p>
        <select
          className="dualSelectInput"
          v-model="selectedTest"
          @input="calculateTrials"
        >
          <option v-for="option in optionsTest" :value="option.value">
            {{ option.text }}
          </option>
        </select>
        <div class="grid">
          <Square v-for="square in squares" :id="`square-${square.id}`" />
        </div>
        <div className="game-controls">
          <div class="play-buttons">
            <button @click="positionCheck()">P</button>
            <button @click="soundCheck()">S</button>
          </div>

          <button className="playButton" @click="startGame($event)">
            Start Game
          </button>
        </div>
        <!--<Controls />-->
      </div>
      <div class="finish-popup">
        <div class="text-inside">
          <div @click="closeFinishPopup()">X</div>
          <p>Correct Location: {{ correctPosition }}</p>
          <p>incorrect Location: {{ incorrectPosition }}</p>
          <p>Correct Sound: {{ correctSound }}</p>
          <p>incorrect Sound: {{ incorrectSound }}</p>
          <p>Selected nBack: {{ nBack.value }}</p>
        </div>
      </div>
    </div>
  </main>
</template>
<style lang="scss">
@import "./assets/base.css";

.main-container {
  width: 100vw;
  height: 100vh;
  align-items: center;
  justify-content: center;

  .playground {
    width: 800px;
    margin: 0 auto;

    .grid {
      margin: 0 auto;
      height: 650px;
      display: grid;
      grid-template-columns: auto auto auto;
      div {
        border: 1px solid black;

        &.active {
          background-color: red;
        }
      }
    }

    .play-buttons {
      display: flex;
      margin-top: 10px;
      button {
        width: 100%;
        height: 60px;
      }
    }
  }
}

.finish-popup {
  width: 100vw;
  height: 100vh;
  display: none;
  text-align: center;
  position: absolute;
  top: 0;
  background-color: rgba(0, 0, 0, 0.3);
  .text-inside {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 500px;
    height: 500px;
    background: #fff;
  }

  &.display-popup {
    display: block;
  }
}
</style>
