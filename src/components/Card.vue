<script setup>
const props = defineProps({
  isActive: Boolean,
  cardType: String
})
</script>

<template>
  <div @click="$emit('cardSelection', cardType)" class="card" :class="{ active: isActive }">
    <div class="card-title">
      <slot name="c_title"></slot>
    </div>
    <div style="display:flex;flex-direction: row;justify-content: flex-end;align-content: flex-end">
      <slot name="c_content"></slot>
    </div> 
  </div>
</template>

<style scoped>
.card {
  border-radius: 1em;
  padding: 2em;
  margin: .5em 0;
  max-width: 800px;

  border: 2px dashed var(--color-text);
  background-color: var(--color-text);
  color: var(--color-background);
}

.card:hover {
  border: 2px dashed var(--color-text);
}

.active {
    border: 2px solid var(--color-border);
    background-color: var(--color-background);
    color: var(--color-text);
}
</style>