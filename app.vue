<template>
  <div label="Microphone">
    <button @click="startAudio" ref="micButton">Tone</button>
    <div id="content">
    </div>
  </div>
</template>

<script lang="ts" setup>
import { UserMedia, Volume, start, FFT, Offline } from 'tone';
const mic = ref() as Ref<UserMedia>
const micFFT = ref() as Ref<FFT>
const micButton = ref()

async function startAudio() {
  try {
    await mic.value.open()
    mic.value.connect(micFFT.value);
    
  } catch (error) {
    console.error(error)
  }
}

onMounted(async () => {
  mic.value = new UserMedia()
  micFFT.value = new FFT()
  
})
</script>
<style>
#content{
width: 100%;
  background-color: black;
  height: 40px;
  border-radius: 20px;
  margin-bottom: 10px;
}
</style>