<template>
  <div label="Microphone">
    <div id="content"></div>
  </div>
</template>

<script lang="ts" setup>
import { UserMedia, FFT } from 'tone';


onMounted(async () => {
  const fft = await new FFT()
  const mic = new UserMedia().connect(fft)
  await mic.open()

  const handler = () => {
    console.log("mic open");
    clearInterval(interval);

    setInterval(() => {
      console.log(fft.getValue());
    }, 5000); // Update every second
  };

  const interval = setInterval(handler, 500); // Check after half-second to ensure microphone has started

});



</script>
<style scoped>
#content {
  width: 100%;
  background-color: black;
  height: 40px;
  border-radius: 20px;
  margin-bottom: 10px;
}
</style>