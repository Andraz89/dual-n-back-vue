<script setup>
const props = defineProps({
  startFunction: Function,
});
</script>

<template>
  <div className="game-controls">
    <div class="play-buttons">
      <button class="positionButton" disabled @click="$emit('answer', 'P')">
        P
      </button>
      <button class="positionSound" disabled @click="$emit('answer', 'S')">
        S
      </button>
    </div>
    <button className="playButton" @click="props.startFunction($event)">
      Start Game
    </button>
  </div>
</template>
