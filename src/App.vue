<script setup>
import { ref, watch } from 'vue';
import Card from './components/Card.vue';
import Toggle from './components/Toggle.vue';

const unit = ref("km");

const tempoOre = ref(0);
const tempoMinuti = ref(30);
const tempoSecondi = ref(0);
const distanza = ref(5);
const passoMinuti = ref(6);
const passoSecondi = ref(0);
const lastChanged = ref('');

const isUpdating = ref(false);
const selectionList = ref(['t', 'd']);

function clamp(value, min, max) {
  const val = parseInt(value || 0);
  return Math.max(min, Math.min(max, isNaN(val) ? 0 : val));
}

function updateField(field) {
  lastChanged.value = field;
}

function validateTempo() {
  let sec = clamp(tempoSecondi.value, 0, Infinity);
  let min = clamp(tempoMinuti.value, 0, Infinity);
  let ore = clamp(tempoOre.value, 0, 999);

  min += Math.floor(sec / 60);
  sec = sec % 60;

  ore += Math.floor(min / 60);
  min = min % 60;

  tempoOre.value = ore;
  tempoMinuti.value = min;
  tempoSecondi.value = sec;
}

function validatePasso() {
  let sec = clamp(passoSecondi.value, 0, Infinity);
  let min = clamp(passoMinuti.value, 0, Infinity);

  min += Math.floor(sec / 60);
  sec = sec % 60;

  passoMinuti.value = clamp(min, 0, 59);  // passoMinuti oltre 59 non ha senso in corsa
  passoSecondi.value = sec;
}

function validateDistanza() {
  distanza.value = Math.max(0, parseFloat(distanza.value || 0));
}

function getTempoTotaleInSecondi() {
  return (
    parseInt(tempoOre.value || 0) * 3600 +
    parseInt(tempoMinuti.value || 0) * 60 +
    parseInt(tempoSecondi.value || 0)
  );
}

function setTempoFromSecondi(totSec) {
  tempoOre.value = Math.floor(totSec / 3600);
  tempoMinuti.value = Math.floor((totSec % 3600) / 60);
  tempoSecondi.value = Math.round(totSec % 60);
}

function setPassoFromSecondi(secPerKm) {
  passoMinuti.value = Math.floor(secPerKm / 60);
  passoSecondi.value = Math.round(secPerKm % 60);
}

function getPassoInSecondi() {
  return parseInt(passoMinuti.value || 0) * 60 + parseInt(passoSecondi.value || 0);
}

function isSelected(selection) {
  return selectionList.value.includes(selection);
}

function selectCard(selection) {
  if (isSelected(selection)) {
    // do nothing
  } else {
    selectionList.value.shift();
    selectionList.value.push(selection);
  }
}

function notInList() {
  if (!selectionList.value.includes('t')) return 't';
  if (!selectionList.value.includes('p')) return 'p';
  return 'd';
}

watch(
  [tempoOre, tempoMinuti, tempoSecondi, distanza, passoMinuti, passoSecondi],
  () => {
    if (isUpdating.value) return;
    isUpdating.value = true;

    const tempoTotale = getTempoTotaleInSecondi();
    const d = parseFloat(distanza.value || 0);
    const passoInSecondi = getPassoInSecondi();

    // calcola passo
    if (notInList() === "p" && d > 0) {
      const secPerKm = tempoTotale / d;
      setPassoFromSecondi(secPerKm);
    }

    // calcola distanza
    if (notInList() === "d") {
      distanza.value = (tempoTotale / passoInSecondi).toFixed(2);
    }

    // calcola tempo
    if (notInList() === "t") {
      const totalSec = d * passoInSecondi;
      setTempoFromSecondi(totalSec);
    }

    isUpdating.value = false;
  }
);

</script>

<template>
  <header>
    <h1>Runcalc</h1>
    <Toggle @toggle-unit="unit = $event"></Toggle>
  </header>
  
  <div style="display: flex; flex-direction: column;width:100%">
    <Card :card-type="'d'" :is-active="isSelected('d')" @card-selection="selectCard">
      <template #c_title>
        <h2>Distance</h2>
      </template>
      <template #c_content>
        <input type="number" inputmode="decimal" step="0.01" min="0.01" max="999.99" v-model="distanza" size="4"
          @input="updateField('distanza')" @blur="validateDistanza" />
        <label class="label-bottom">{{ unit }}</label>
      </template>
    </Card>

    <Card :card-type="'t'" :is-active="isSelected('t')" @card-selection="selectCard">
      <template #c_title>
        <h2>Time</h2>
      </template>
      <template #c_content>
        <input type="number" inputmode="numeric" min="0" max="72" size="2" v-model="tempoOre" @input="updateField('tempo')" @blur="validateTempo" />
        <label class="label-bottom">h</label>
        <input type="number" inputmode="numeric" min="0" max="59" size="2" v-model="tempoMinuti" @input="updateField('tempo')"
          @blur="validateTempo" />
        <label class="label-top">'</label>
        <input type="number" inputmode="numeric" min="0" max="59" size="2" v-model="tempoSecondi" @input="updateField('tempo')"
          @blur="validateTempo" />
        <label class="label-top">"</label>
      </template>
    </Card>

    <Card :card-type="'p'" :is-active="isSelected('p')" @card-selection="selectCard">
      <template #c_title>
        <h2>Pace</h2>
      </template>
      <template #c_content>
        <input type="number" inputmode="numeric" min="0" max="59" size="1" v-model="passoMinuti" @input="updateField('passo')"
          @blur="validatePasso" />
        <label class="label-top">'</label>

        <input type="number" inputmode="numeric" min="0" max="59" size="2" v-model="passoSecondi" @input="updateField('passo')"
          @blur="validatePasso" />
        <label class="label-top">"</label>
        <label class="label-bottom">/{{ unit }}</label>
      </template>
    </Card>
  </div>

  <footer>
    Runcalc v.1 2025
  </footer>
</template>

<style scoped>

header {
  display:flex;
  flex-direction: row;
  justify-content: space-between;
  width:100%;
  align-items: baseline;
}

footer {
  text-align: center;
}

input:invalid {
  border: solid red 3px;
}

input {
  background-color: inherit;
  color: inherit;
  border: 0;
  border-bottom: 1px solid var(--color-border);
  font-size: 1.5rem;
  font-weight: bold;
  text-align: right;
  margin: 0 8px;
}

h1, h2 {
  font-weight: bold;
}

input[type=number]::-webkit-outer-spin-button,
input[type=number]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type=number] {
  -moz-appearance: textfield;
}

.label-bottom {
  display: flex;
  align-self: flex-end;
}

.label-top {
  display: flex;
  align-self: flex-start;
}

.label-bottom,
.label-top {
  font-size: 2em;
  font-weight: bold;
}

</style>
