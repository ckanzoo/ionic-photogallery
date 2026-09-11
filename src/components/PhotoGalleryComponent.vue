<template>
  <ion-card>
    <ion-card-header>
      <ion-card-title>Photo Gallery</ion-card-title>
    </ion-card-header>
    <ion-card-content>
      <div v-if="photos.length === 0" class="empty-gallery">
        <p>No pictures yet.</p>
        <p>Take a picture using the camera.</p>
      </div>
      <ion-grid v-else>
        <ion-row>
          <ion-col
            v-for="(photo, index) in photos"
            :key="index"
            size="6"
            size-md="4"
          >
            <ion-card>
              <ion-img :src="photo" class="gallery-image" />
            </ion-card>
          </ion-col>
        </ion-row>
      </ion-grid>
    </ion-card-content>
  </ion-card>
</template>

<script setup lang="ts">
import {
  IonCard,
  IonCardContent,
  IonCardHeader,
  IonCardTitle,
  IonGrid,
  IonRow,
  IonCol,
  IonImg,
} from "@ionic/vue";

defineProps<{ photos: string[] }>();
</script>

<style scoped>
.empty-gallery {
  text-align: center;
  padding: 30px;
  color: #777;
}

.gallery-image {
  width: 100%;
  height: 180px;
}

.gallery-image::part(image) {
  object-fit: cover;
  transform: scaleX(-1); /* I-flip pabalik horizontally ang na-mirror na picture */
}
</style>