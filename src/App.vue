<template>
  <div class="app">
    <button :class="`mic ${isRecording ? 'recording' : ''}`" @click="ToggleMic">
      <font-awesome-icon :icon="isRecording ? 'stop' : 'microphone'" />
      {{ isRecording ? 'Stop' : 'Record' }}
    </button>
    <div class="transcript">{{ transcript }}</div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const transcript = ref("");
const isRecording = ref(false);

const Recognition = window.SpeechRecognition || window.webkitSpeechRecognition;
const sr = new Recognition();

const ToggleMic = () => {
  if (isRecording.value) sr.stop();
  else sr.start();
};

onMounted(() => {
  sr.continuous = true;
  sr.interimResults = true;

  sr.onstart = () => {
    isRecording.value = true;
  };

  sr.onend = () => {
    isRecording.value = false;
  };

  sr.onresult = (evt) => {
    for (let i = 0; i < evt.results.length; i++) {
      const result = evt.results[i];

      if (result.isFinal) CheckForCommand(result);
    }

    const t = Array.from(evt.results)
      .map((result) => result[0])
      .map((result) => result.transcript)
      .join("");

    transcript.value = t;
  };

  const CheckForCommand = (result) => {
    const t = result[0].transcript.toLowerCase();
    if (t.includes("stop")) {
      sr.stop();
    } else if (
      t.includes("what is the time") ||
      t.includes("what's the time")
    ) {
      sr.stop();
      alert(new Date().toLocaleTimeString());
      setTimeout(() => sr.start, 100);
    }
  };
});
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Fira Sans", sans-serif;
}

body {
  background-color: #07072c;
  color: #fff;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  padding: 20px;
  text-align: center;
}

.mic {
  background-color: #1a73e8;
  color: #fff;
  border: none;
  padding: 15px 25px;
  font-size: 1.2em;
  border-radius: 50px;
  cursor: pointer;
  transition: background-color 0.3s;
  display: flex;
  align-items: center;
  gap: 10px;
}

.mic.recording {
  background-color: #d32f2f;
}

.mic:hover {
  background-color: #005cbf;
}

.transcript {
  margin-top: 20px;
  padding: 10px;
  background-color: #202040;
  border-radius: 10px;
  max-width: 600px;
  width: 100%;
  word-wrap: break-word;
  font-size: 1.1em;
}
</style>
