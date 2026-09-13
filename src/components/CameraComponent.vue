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

const takePicture = async () => {
  errorMessage.value = "";
  try {
    const photo = await Camera.getPhoto({
      quality: 75,
      allowEditing: false,
      resultType: CameraResultType.Base64,
      source: CameraSource.Camera,
      saveToGallery: true,
    });

    if (photo.base64String) {
      const imageUri = `data:image/jpeg;base64,${photo.base64String}`;
      emit("photoCaptured", imageUri);
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