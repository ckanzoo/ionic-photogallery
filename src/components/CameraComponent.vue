<template>
  <div class="camera-wrapper">
    <ion-button shape="round" class="shutter-btn" @click="takePicture">
      <ion-icon slot="start" :icon="cameraIcon" />
      Take Photo
    </ion-button>
    <ion-text v-if="errorMessage" color="danger" class="error-msg">
      <p>{{ errorMessage }}</p>
    </ion-text>
  </div>
</template>

<script setup lang="ts">
import { IonButton, IonIcon, IonText } from "@ionic/vue";
import { camera as cameraIcon } from "ionicons/icons";
import { Camera, CameraResultType, CameraSource } from "@capacitor/camera";
import { ref } from "vue";

const errorMessage = ref("");
const emit = defineEmits<{ (event: "photoCaptured", photo: string): void }>();

// Tunog ng mechanical shutter gamit ang built-in Web Audio API
const playShutterSound = () => {
  try {
    const AudioContextClass = window.AudioContext || (window as unknown as { webkitAudioContext: typeof AudioContext }).webkitAudioContext;
    const ctx = new AudioContextClass();

    // 1st click
    const osc1 = ctx.createOscillator();
    const gain1 = ctx.createGain();
    osc1.type = "triangle";
    osc1.frequency.setValueAtTime(140, ctx.currentTime);
    osc1.frequency.exponentialRampToValueAtTime(30, ctx.currentTime + 0.04);
    gain1.gain.setValueAtTime(0.7, ctx.currentTime);
    gain1.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.04);
    osc1.connect(gain1);
    gain1.connect(ctx.destination);
    osc1.start();
    osc1.stop(ctx.currentTime + 0.04);

    // 2nd shutter snap
    setTimeout(() => {
      const osc2 = ctx.createOscillator();
      const gain2 = ctx.createGain();
      osc2.type = "square";
      osc2.frequency.setValueAtTime(220, ctx.currentTime);
      osc2.frequency.exponentialRampToValueAtTime(40, ctx.currentTime + 0.05);
      gain2.gain.setValueAtTime(0.5, ctx.currentTime);
      gain2.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.05);
      osc2.connect(gain2);
      gain2.connect(ctx.destination);
      osc2.start();
      osc2.stop(ctx.currentTime + 0.05);
    }, 45);
  } catch (err) {
    console.warn("Audio playback not supported or blocked", err);
  }
};

// Retro orange quartz timestamp renderer
const addRetroTimestamp = (base64Data: string): Promise<string> => {
  return new Promise((resolve) => {
    const img = new Image();
    img.crossOrigin = "anonymous";
    img.onload = () => {
      const canvas = document.createElement("canvas");
      canvas.width = img.width;
      canvas.height = img.height;
      const ctx = canvas.getContext("2d");

      if (!ctx) {
        resolve(base64Data);
        return;
      }

      ctx.drawImage(img, 0, 0);

      // Petsa format: '26 09 13
      const now = new Date();
      const year = now.getFullYear().toString().slice(-2);
      const month = String(now.getMonth() + 1).padStart(2, "0");
      const day = String(now.getDate()).padStart(2, "0");
      const dateText = `'${year} ${month} ${day}`;

      // Styling: Courier New orange digital quartz
      const fontSize = Math.max(28, Math.floor(canvas.width * 0.038));
      ctx.font = `bold ${fontSize}px "Courier New", monospace`;
      ctx.fillStyle = "#ff8c00";
      ctx.shadowColor = "rgba(255, 69, 0, 0.85)";
      ctx.shadowBlur = 6;

      const padding = Math.floor(canvas.width * 0.04);
      const textMetrics = ctx.measureText(dateText);
      const x = canvas.width - textMetrics.width - padding;
      const y = canvas.height - padding;

      ctx.fillText(dateText, x, y);
      resolve(canvas.toDataURL("image/jpeg", 0.92));
    };

    img.onerror = () => resolve(base64Data);
    img.src = base64Data;
  });
};

const takePicture = async () => {
  errorMessage.value = "";
  try {
    playShutterSound();

    const photo = await Camera.getPhoto({
      quality: 85,
      allowEditing: false,
      resultType: CameraResultType.Base64,
      source: CameraSource.Camera,
      saveToGallery: false,
    });

    if (photo.base64String) {
      const rawUri = `data:image/jpeg;base64,${photo.base64String}`;
      const stampedUri = await addRetroTimestamp(rawUri);
      emit("photoCaptured", stampedUri);
    }
  } catch (error) {
    console.error(error);
    errorMessage.value = "Capture cancelled or failed.";
  }
};
</script>

<style scoped>
.camera-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.shutter-btn {
  --background: #0a84ff;
  --border-radius: 24px;
  --padding-start: 22px;
  --padding-end: 22px;
  height: 42px;
  font-size: 0.95rem;
  font-weight: 600;
  letter-spacing: 0.2px;
  margin: 0;
  box-shadow: 0 4px 14px rgba(10, 132, 255, 0.4);
}

.error-msg {
  position: absolute;
  top: -30px;
  font-size: 0.75rem;
  background: rgba(0, 0, 0, 0.75);
  padding: 2px 8px;
  border-radius: 6px;
}
</style>