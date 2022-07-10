<script setup>
const props = defineProps({
  correctPosition: Number,
  incorrectPosition: Number,
  correctSound: Number,
  incorrectSound: Number,
  nBack: Number,
  closeFinishPopup: Function,
});
</script>
<template>
  <div class="finish-popup">
    <div class="text-inside">
      <div @click="closeFinishPopup()">X</div>
      <p>Correct Location: {{ props.correctPosition }}</p>
      <p>incorrect Location: {{ props.incorrectPosition }}</p>
      <p>Correct Sound: {{ props.correctSound }}</p>
      <p>incorrect Sound: {{ props.incorrectSound }}</p>
      <p>Selected nBack: {{ props.nBack }}</p>
    </div>
  </div>
</template>
