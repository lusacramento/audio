<template>
  <div>
    <button ref="startButton" disabled @click.prevent="startRecord">Gravar</button>
    Meter: {{ meterValue }}
  </div>
</template>

<script lang="ts" setup>
import { UserMedia, Meter } from 'tone';

const startButton = ref()
const meterValue = ref()


onMounted(() => {
  startButton.value.disabled = false
})


function startRecord() {
  console.log("start record");
  startButton.value.disabled = true


  const meter = new Meter()
  const mic = new UserMedia()
  mic.connect(meter)
  mic.open().then(() => {
    // promise resolves when input is available
    console.log("mic open");
    // print the incoming mic levels in decibels
    setInterval(() => meterValue.value = meter.getValue(), 200);
  }).catch(e => {
    // promise is rejected when the user doesn't have or allow mic access
    console.log("mic not open");
  });





}

function stopRecord() {
  stopButton.value.disabled = true
  startButton.value.disabled = false
}

</script>
<style scoped>
/* #content{
width: 100%;
  background-color: black;
  height: 40px;
  border-radius: 20px;
  margin-bottom: 10px;
}
*/
</style>