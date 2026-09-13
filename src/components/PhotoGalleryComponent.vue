<template>
  <div class="gallery-container">
    <div v-if="photos.length === 0" class="empty-state">
      <div class="icon-circle">
        <ion-icon :icon="activeSegment === 'favorites' ? heartDislikeOutline : imagesOutline" class="empty-icon" />
      </div>
      <h3 class="empty-title">{{ activeSegment === 'favorites' ? 'No Favorites Yet' : 'No Photos Yet' }}</h3>
      <p class="empty-desc">
        {{ activeSegment === 'favorites' ? 'Tap the heart icon on any photo to save it here.' : 'Capture your first memory using the camera below.' }}
      </p>
    </div>

    <ion-grid v-else class="grid-container">
      <ion-row>
        <ion-col
          v-for="photo in photos"
          :key="photo.id"
          size="6"
          size-md="4"
          size-lg="3"
        >
          <div class="photo-card" @click="$emit('previewPhoto', photo)">
            <img
              :src="photo.data"
              class="photo-img"
              :style="{ filter: photo.filter || 'none' }"
            />

            <!-- Date Stamp -->
            <div class="date-overlay">
              <span>{{ photo.date }}</span>
            </div>

            <!-- Favorite Heart Button -->
            <button
              class="badge-btn favorite-btn"
              type="button"
              @click.stop="$emit('toggleFavorite', photo.id)"
              title="Favorite"
            >
              <ion-icon :icon="photo.isFavorite ? heart : heartOutline" :class="{ 'is-fav': photo.isFavorite }" />
            </button>

            <!-- Delete Button -->
            <button
              class="badge-btn delete-btn"
              type="button"
              @click.stop="$emit('deletePhoto', photo.id)"
              title="Delete"
            >
              <ion-icon :icon="trashOutline" />
            </button>
          </div>
        </ion-col>
      </ion-row>
    </ion-grid>
  </div>
</template>

<script setup lang="ts">
import { IonGrid, IonRow, IonCol, IonIcon } from "@ionic/vue";
import { imagesOutline, trashOutline, heart, heartOutline, heartDislikeOutline } from "ionicons/icons";
import type { GalleryItem } from "@/views/HomePage.vue";

defineProps<{
  photos: GalleryItem[];
  activeSegment: "all" | "favorites";
}>();

defineEmits<{
  (event: "deletePhoto", id: string): void;
  (event: "toggleFavorite", id: string): void;
  (event: "previewPhoto", photo: GalleryItem): void;
}>();
</script>

<style scoped>
.gallery-container {
  padding: 12px;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 50vh;
  text-align: center;
}

.icon-circle {
  width: 80px;
  height: 80px;
  background: #1c1c1e;
  border: 1px solid #2c2c2e;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 14px;
}

.empty-icon {
  font-size: 38px;
  color: #636366;
}

.empty-title {
  font-size: 1.15rem;
  font-weight: 600;
  color: #ffffff;
  margin: 0 0 6px 0;
}

.empty-desc {
  font-size: 0.85rem;
  color: #8e8e93;
  max-width: 240px;
  margin: 0;
  line-height: 1.4;
}

.photo-card {
  position: relative;
  border-radius: 16px;
  overflow: hidden;
  background-color: #1c1c1e;
  border: 1px solid rgba(255, 255, 255, 0.08);
  aspect-ratio: 1 / 1;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.4);
  cursor: pointer;
  transition: transform 0.2s ease;
}

.photo-card:active {
  transform: scale(0.97);
}

.photo-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  transition: filter 0.2s ease;
}

.date-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 16px 8px 6px 8px;
  background: linear-gradient(to top, rgba(0, 0, 0, 0.75), transparent);
  font-size: 0.7rem;
  color: #f2f2f7;
  font-weight: 500;
  pointer-events: none;
}

.badge-btn {
  position: absolute;
  top: 8px;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 5;
  transition: transform 0.15s ease;
}

.badge-btn:active {
  transform: scale(0.85);
}

.favorite-btn {
  left: 8px;
  color: #ffffff;
}

.is-fav {
  color: #ff375f !important;
}

.delete-btn {
  right: 8px;
  color: #ff453a;
}
</style>