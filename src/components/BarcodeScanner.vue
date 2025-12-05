<template>
  <div>
    <h3>Barcode / QR Scanner (Camera + Upload)</h3>

    <video ref="videoRef" class="preview"></video>

    <p v-if="result.text">
      Detected: {{ result.text }} <br />
      <strong>Type:</strong> {{ result.format }}
    </p>

    <hr />

    <!-- UPLOAD IMAGE -->
    <h4>Scan dari Gambar</h4>
    <input type="file" accept="image/*" @change="onFileChange" />

    <div v-if="preview" class="preview-box">
      <img :src="preview" class="preview" />
    </div>

    <p v-if="uploadResult.text">
      ✔ Hasil Upload: {{ uploadResult.text }} <br />
      <strong>Jenis:</strong> {{ uploadResult.format }}
    </p>

    <p v-if="uploadError" class="error">{{ uploadError }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { BrowserMultiFormatReader } from "@zxing/browser";
import { BarcodeFormat, DecodeHintType } from "@zxing/library";

const videoRef = ref(null);
const result = ref({ text: "", format: "" });
const uploadResult = ref({ text: "", format: "" });
const uploadError = ref("");
const preview = ref("");

let scanner;
const formatNames = {
  [BarcodeFormat.AZTEC]: "AZTEC",
  [BarcodeFormat.CODABAR]: "CODABAR",
  [BarcodeFormat.CODE_39]: "CODE_39",
  [BarcodeFormat.CODE_93]: "CODE_93",
  [BarcodeFormat.CODE_128]: "CODE_128",
  [BarcodeFormat.DATA_MATRIX]: "DATA_MATRIX",
  [BarcodeFormat.EAN_8]: "EAN_8",
  [BarcodeFormat.EAN_13]: "EAN_13",
  [BarcodeFormat.ITF]: "ITF",
  [BarcodeFormat.MAXICODE]: "MAXICODE",
  [BarcodeFormat.PDF_417]: "PDF_417",
  [BarcodeFormat.QR_CODE]: "QR_CODE",
  [BarcodeFormat.RSS_14]: "RSS_14",
  [BarcodeFormat.RSS_EXPANDED]: "RSS_EXPANDED",
  [BarcodeFormat.UPC_A]: "UPC_A",
  [BarcodeFormat.UPC_E]: "UPC_E",
  [BarcodeFormat.UPC_EAN_EXTENSION]: "UPC_EAN_EXTENSION"
};

const hints = new Map();
hints.set(DecodeHintType.POSSIBLE_FORMATS, [
  BarcodeFormat.QR_CODE,
  BarcodeFormat.CODE_128,
  BarcodeFormat.CODE_39,
  BarcodeFormat.EAN_13,
  BarcodeFormat.EAN_8,
  BarcodeFormat.PDF_417
]);

onMounted(() => {
  scanner = new BrowserMultiFormatReader(hints);

  scanner.decodeFromVideoDevice(
    null, // kamera default
    videoRef.value,
    (scanResult, err) => {
      if (scanResult) {
        result.value = {
  text: scanResult.getText(),
  format: formatNames[scanResult.getBarcodeFormat()] || "UNKNOWN"
};

        console.log("Camera Detected:", result.value);
      }
    }
  );
});

onBeforeUnmount(() => scanner?.reset());

const onFileChange = async (event) => {
  const file = event.target.files[0];
  if (!file) return;

  uploadResult.value = { text: "", format: "" };
  uploadError.value = "";

  const imgURL = URL.createObjectURL(file);
  preview.value = imgURL;

  try {
    const decoder = new BrowserMultiFormatReader(hints);
    const scanResult = await decoder.decodeFromImageUrl(imgURL);

    uploadResult.value = {
  text: scanResult.getText(),
  format: formatNames[scanResult.getBarcodeFormat()] || "UNKNOWN"
};


    console.log("Image Detected:", uploadResult.value);
  } catch (err) {
    uploadError.value = "❌ Barcode tidak ditemukan dalam gambar.";
    console.error(err);
  }
};
</script>

<style scoped>
.preview {
  width: 100%;
  max-width: 350px;
  border-radius: 10px;
  border: 2px solid #444;
}
.preview-box {
  margin-top: 10px;
}
.error {
  color: red;
}
</style>
