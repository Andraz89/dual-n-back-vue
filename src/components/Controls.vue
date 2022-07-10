<script setup>
const props = defineProps({
  startFunction: Function,
});
</script>

<template>
  <div className="game-controls">
    <div class="play-buttons">
      <button @click="$emit('answer', 'P')">P</button>
      <button @click="$emit('answer', 'S')">S</button>
    </div>
    <button className="playButton" @click="props.startFunction($event)">
      Start Game
    </button>
  </div>
</template>
