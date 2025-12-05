<template>
  <div>
    <h3>QR Generator</h3>
    <input v-model="text" placeholder="Text to QR" class="input" />
    <button @click="generate">Generate</button>

    <canvas ref="qrCanvas"></canvas>
  </div>
</template>

<script setup>
import { ref } from "vue";
import QRCode from "qrcode";

const text = ref("Hello World");
const qrCanvas = ref(null);

const generate = () => {
  QRCode.toCanvas(qrCanvas.value, text.value, {
    width: 200,
    margin: 2,
  });
};
</script>

<style scoped>
.input {
  padding: 8px;
  margin-bottom: 10px;
}
</style>
