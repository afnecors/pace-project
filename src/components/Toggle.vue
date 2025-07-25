<script setup>
import { ref } from 'vue'

const unit = ref('km');

const emit = defineEmits(['toggleUnit']);

function buttonClick(newUnit) {
  unit.value = newUnit;
  emit('toggleUnit', newUnit)
}

</script>

<template>
  <div class="toggle-container">
    <div @click="buttonClick('km')" :class="{active: unit === 'km', 'toggle-item': true}">km</div>
    <div @click="buttonClick('mi')" :class="{active: unit === 'mi', 'toggle-item': true}">mi</div>
  </div>
</template>

<style scoped>
.toggle-container {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.active {
  font-weight: bold !important;
}

.toggle-item {
  width: 2em;
  height: 2em;
  font-weight: normal;
  cursor: pointer;
}
</style>