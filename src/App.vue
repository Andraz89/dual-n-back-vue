<script setup>
import { ref, onMounted } from "vue";
import Square from "./components/Square.vue";
import Controls from "./components/Controls.vue";
import EndingStats from "./components/EndingStats.vue";

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
let showEndStat = ref(false);

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

function resetScore() {
  correctPosition = ref(0);
  incorrectPosition = ref(0);
  correctSound = ref(0);
  incorrectSound = ref(0);
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
    console.log(activate);
    count.value++;
  }, 3000);
}

/*function positionCheck() {
  activate.value[gameProgression.value].posCorrect = true;
}

function soundCheck() {
  activate.value[gameProgression.value].soundCorrect = true;
}*/

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
  showEndStat.value = true;
  clearInterval(showSquare);
  createChallenge();
}

function closeFinishPopup() {
  showEndStat.value = false;
  resetScore();
  document.querySelector(".finish-popup").classList.remove("display-popup");
  document.querySelector(".playButton").removeAttribute("disabled");
  count.value = 0;
}

function writeAnswer(btn) {
  if (btn == "P") {
    activate.value[gameProgression.value].posCorrect = true;
  } else if (btn == "S") {
    activate.value[gameProgression.value].soundCorrect = true;
  }
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
        <Controls
          :activeObject="activate"
          :gameProgression="gameProgression"
          :startFunction="startGame"
          @answer="writeAnswer"
        />
      </div>
      <EndingStats
        v-if="showEndStat"
        :correctPosition="correctPosition"
        :incorrectPosition="incorrectPosition"
        :correctSound="correctSound"
        :incorrectSound="incorrectSound"
        :nBack="nBack"
        :closeFinishPopup="closeFinishPopup"
      />
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
  //display: none;
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
