<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar class="custom-toolbar">
        <ion-title>
          <div class="header-title-container">
            <span class="main-title">Obscura</span>
            <span class="photo-counter">{{ counterText }}</span>
          </div>
        </ion-title>
      </ion-toolbar>

      <ion-toolbar class="segment-toolbar">
        <ion-segment v-model="selectedSegment" mode="ios" class="custom-segment">
          <ion-segment-button value="all">
            <ion-label>All Photos</ion-label>
          </ion-segment-button>
          <ion-segment-button value="favorites">
            <ion-label>Favorites ({{ favoriteCount }})</ion-label>
          </ion-segment-button>
        </ion-segment>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="gallery-content">
      <div class="content-wrapper">
        <PhotoGalleryComponent
          :photos="filteredPhotos"
          :active-segment="selectedSegment"
          @delete-photo="removePhoto"
          @toggle-favorite="toggleFavorite"
          @preview-photo="openPreview"
        />
      </div>

      <!-- Frosted Glass Bottom Action Bar -->
      <div class="bottom-action-bar">
        <div class="frosted-pill">
          <CameraComponent @photo-captured="addPhoto" />
        </div>
      </div>

      <!-- Full-Screen Lightbox Modal -->
      <ion-modal :is-open="isPreviewOpen" @didDismiss="closePreview" class="preview-modal">
        <div class="preview-backdrop">
          <!-- Top Controls -->
          <div class="modal-top-bar">
            <button class="top-action-btn" @click="sharePhoto" title="Share Photo">
              <ion-icon :icon="shareOutline" />
            </button>
            <button class="top-action-btn" @click="downloadToDevice" title="Save to Device">
              <ion-icon :icon="downloadOutline" />
            </button>
            <button class="top-action-btn close-btn" @click="closePreview">✕</button>
          </div>
          
          <!-- Large Image Display -->
          <div class="modal-image-container">
            <img
              v-if="previewTarget"
              :src="previewTarget.data"
              class="full-image"
              :style="{ filter: previewTarget.filter || 'none' }"
            />
          </div>

          <!-- Filter Selection Presets -->
          <div v-if="previewTarget" class="filters-panel">
            <span class="filters-label">Film Presets</span>
            <div class="filters-row">
              <button
                v-for="item in filterPresets"
                :key="item.name"
                class="filter-chip"
                :class="{ active: (previewTarget.filter || 'none') === item.css }"
                @click="applyFilter(item.css)"
              >
                {{ item.name }}
              </button>
            </div>
          </div>

          <!-- Toast Feedback & Metadata -->
          <div v-if="previewTarget" class="modal-caption">
            <span>{{ previewTarget.date }}</span>
            <span v-if="statusMessage" class="status-badge">{{ statusMessage }}</span>
          </div>
        </div>
      </ion-modal>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonSegment,
  IonSegmentButton,
  IonLabel,
  IonModal,
  IonIcon,
} from "@ionic/vue";
import { downloadOutline, shareOutline } from "ionicons/icons";
import { ref, computed, onMounted } from "vue";
import CameraComponent from "@/components/CameraComponent.vue";
import PhotoGalleryComponent from "@/components/PhotoGalleryComponent.vue";

export interface GalleryItem {
  id: string;
  data: string;
  date: string;
  isFavorite: boolean;
  filter?: string;
}

const STORAGE_KEY = "user_captured_photos_v2";
const photos = ref<GalleryItem[]>([]);
const selectedSegment = ref<"all" | "favorites">("all");
const statusMessage = ref("");

const filterPresets = [
  { name: "Normal", css: "none" },
  { name: "B&W", css: "grayscale(100%)" },
  { name: "Sepia", css: "sepia(80%) contrast(110%)" },
  { name: "Warm", css: "sepia(30%) saturate(140%)" },
  { name: "Cool", css: "hue-rotate(180deg) saturate(110%)" },
  { name: "Vintage", css: "contrast(120%) brightness(90%) sepia(40%)" },
];

const isPreviewOpen = ref(false);
const previewTarget = ref<GalleryItem | null>(null);

onMounted(() => {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (saved) {
    try {
      photos.value = JSON.parse(saved);
    } catch (e) {
      console.error("Failed to parse photos", e);
    }
  }
});

const persistPhotos = () => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(photos.value));
};

const addPhoto = (photoUri: string) => {
  const newItem: GalleryItem = {
    id: Date.now().toString(),
    data: photoUri,
    date: new Date().toLocaleDateString("en-US", {
      month: "short",
      day: "numeric",
      hour: "2-digit",
      minute: "2-digit",
    }),
    isFavorite: false,
    filter: "none",
  };
  photos.value.unshift(newItem);
  persistPhotos();
};

const removePhoto = (id: string) => {
  photos.value = photos.value.filter((p) => p.id !== id);
  persistPhotos();
  if (previewTarget.value?.id === id) {
    closePreview();
  }
};

const toggleFavorite = (id: string) => {
  const target = photos.value.find((p) => p.id === id);
  if (target) {
    target.isFavorite = !target.isFavorite;
    persistPhotos();
  }
};

const openPreview = (photo: GalleryItem) => {
  previewTarget.value = photo;
  statusMessage.value = "";
  isPreviewOpen.value = true;
};

const closePreview = () => {
  isPreviewOpen.value = false;
  previewTarget.value = null;
  statusMessage.value = "";
};

const applyFilter = (filterCss: string) => {
  if (previewTarget.value) {
    previewTarget.value.filter = filterCss;
    const target = photos.value.find((p) => p.id === previewTarget.value?.id);
    if (target) {
      target.filter = filterCss;
      persistPhotos();
    }
  }
};

// I-save ang litrato sa Downloads/Camera Roll
const downloadToDevice = () => {
  if (!previewTarget.value) return;
  try {
    const link = document.createElement("a");
    link.href = previewTarget.value.data;
    link.download = `obscura_${Date.now()}.jpg`;
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    statusMessage.value = "Saved to Device!";
    setTimeout(() => {
      statusMessage.value = "";
    }, 2500);
  } catch (err) {
    console.error("Save failed", err);
    statusMessage.value = "Failed to save.";
  }
};

// Web Share API (native share sa iOS para sa AirDrop / Photos)
const sharePhoto = async () => {
  if (!previewTarget.value) return;
  try {
    if (navigator.share) {
      const res = await fetch(previewTarget.value.data);
      const blob = await res.blob();
      const file = new File([blob], `obscura_${Date.now()}.jpg`, { type: "image/jpeg" });
      await navigator.share({
        files: [file],
        title: "Obscura Capture",
      });
    } else {
      downloadToDevice();
    }
  } catch (err) {
    console.warn("Share sheet closed or unsupported", err);
  }
};

const filteredPhotos = computed(() => {
  if (selectedSegment.value === "favorites") {
    return photos.value.filter((p) => p.isFavorite);
  }
  return photos.value;
});

const favoriteCount = computed(() => photos.value.filter((p) => p.isFavorite).length);

const counterText = computed(() => {
  const count = photos.value.length;
  return `${count} ${count === 1 ? "Photo" : "Photos"}`;
});
</script>

<style scoped>
.custom-toolbar {
  --background: rgba(28, 28, 30, 0.95);
  --color: #ffffff;
  backdrop-filter: blur(20px);
}

.header-title-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 4px 0;
}

.main-title {
  font-size: 1.1rem;
  font-weight: 700;
  letter-spacing: 0.5px;
}

.photo-counter {
  font-size: 0.75rem;
  color: #8e8e93;
  font-weight: 500;
}

.segment-toolbar {
  --background: rgba(28, 28, 30, 0.95);
  padding: 0 16px 8px 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.08);
}

.custom-segment {
  --background: rgba(118, 118, 128, 0.24);
}

.gallery-content {
  --background: #0f0f11;
}

.content-wrapper {
  padding-bottom: 120px;
}

.bottom-action-bar {
  position: fixed;
  bottom: 24px;
  left: 0;
  right: 0;
  display: flex;
  justify-content: center;
  z-index: 1000;
  pointer-events: none;
}

.frosted-pill {
  pointer-events: auto;
  padding: 6px 12px;
  background: rgba(28, 28, 30, 0.65);
  backdrop-filter: blur(24px) saturate(180%);
  -webkit-backdrop-filter: blur(24px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-radius: 40px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.45);
}

/* Lightbox Modal */
.preview-backdrop {
  position: relative;
  width: 100%;
  height: 100%;
  background: rgba(12, 12, 14, 0.98);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 50px 16px 30px 16px;
}

.modal-top-bar {
  position: absolute;
  top: 18px;
  left: 18px;
  right: 18px;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  z-index: 10;
}

.top-action-btn {
  background: rgba(255, 255, 255, 0.15);
  color: #fff;
  border: none;
  width: 38px;
  height: 38px;
  border-radius: 50%;
  font-size: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  backdrop-filter: blur(10px);
}

.top-action-btn:active {
  transform: scale(0.9);
}

.modal-image-container {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  max-height: 55vh;
}

.full-image {
  max-width: 90%;
  max-height: 100%;
  object-fit: contain;
  border-radius: 16px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.7);
  transition: filter 0.25s ease;
}

/* Filters Bar */
.filters-panel {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 12px;
}

.filters-label {
  font-size: 0.8rem;
  color: #8e8e93;
  text-transform: uppercase;
  letter-spacing: 0.8px;
  margin-bottom: 8px;
}

.filters-row {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding: 4px 8px;
  max-width: 100%;
}

.filter-chip {
  background: #1c1c1e;
  color: #c7c7cc;
  border: 1px solid rgba(255, 255, 255, 0.12);
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 0.8rem;
  font-weight: 500;
  cursor: pointer;
  white-space: nowrap;
  transition: all 0.2s ease;
}

.filter-chip.active {
  background: #0a84ff;
  color: #ffffff;
  border-color: #0a84ff;
  box-shadow: 0 0 12px rgba(10, 132, 255, 0.5);
}

.modal-caption {
  margin-top: 10px;
  color: #636366;
  font-size: 0.8rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.status-badge {
  color: #30d158;
  font-weight: 600;
}
</style>