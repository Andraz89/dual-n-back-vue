<script setup>
defineProps({
  activeObject: Object,
  gameProgression: Number,
});

function positionCheck(props) {
  console.log(props);
  //activeObject.value[gameProgression.value].posCorrect = true;
}

function soundCheck() {
  //activeObject.value[gameProgression.value].soundCorrect = true;
}
</script>

<template>
  <div className="game-controls">
    <div class="play-buttons">
      <button @click="positionCheck()">P</button>
      <button @click="soundCheck()">S</button>
    </div>
    <button className="playButton" @click="startGame($event)">
      Start Game
    </button>
  </div>
</template>
