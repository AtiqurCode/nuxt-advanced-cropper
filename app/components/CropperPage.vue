<template>
  <div>
    <header class="mb-8 text-center">
      <h1 class="text-3xl font-bold mb-2">Advanced Image Cropper</h1>
      <p class="text-gray-500">Upload, crop, and download your images with precision</p>
    </header>

    <div class="flex flex-col lg:flex-row gap-8">
      <!-- Left panel with controls -->
      <div class="w-full lg:w-1/3">
        <UCard class="mb-6">
          <template #header>
            <div class="flex items-center justify-between">
              <h2 class="text-xl font-semibold">Upload Image</h2>
              <UButton
                v-if="hasImage"
                color="red"
                variant="ghost"
                icon="i-heroicons-trash"
                size="sm"
                @click="resetImage"
              />
            </div>
          </template>

          <div class="mb-4">
            <input
              type="file"
              accept="image/*"
              @change="handleFileUpload"
              class="block w-full text-sm text-gray-500
                file:mr-4 file:py-2 file:px-4
                file:rounded-full file:border-0
                file:text-sm file:font-semibold
                file:bg-primary-50 file:text-primary-700
                hover:file:bg-primary-100
                cursor-pointer"
            />
          </div>
        </UCard>

        <UCard v-if="hasImage" class="mb-6">
          <template #header>
            <h2 class="text-xl font-semibold">Adjustments</h2>
          </template>
          
          <div class="space-y-4">
            <AspectRatioControl
              v-model="selectedRatio"
              :ratios="aspectRatios"
            />
            
            <RotateFlipControl
              @rotate-left="rotateLeft"
              @rotate-right="rotateRight"
              @flip-horizontal="flipHorizontal"
              @flip-vertical="flipVertical"
            />

            <ZoomControl
              v-model="zoom"
              @update:model-value="updateZoom"
            />
          </div>
        </UCard>

        <UCard v-if="hasImage && croppedImage">
          <template #header>
            <h2 class="text-xl font-semibold">Actions</h2>
          </template>

          <div class="space-y-4">
            <UButton
              color="primary"
              block
              icon="i-heroicons-arrow-down-tray"
              @click="downloadImage"
            >
              Download Image
            </UButton>
          </div>
        </UCard>
      </div>

      <!-- Right panel with cropper and preview -->
      <div class="w-full lg:w-2/3">
        <div v-if="!hasImage" class="cropper-placeholder">
          <UCard class="h-96 flex items-center justify-center">
            <div class="text-center">
              <UIcon name="i-heroicons-photo" class="text-5xl text-gray-300 mb-2" />
              <p class="text-gray-500">Upload an image to start cropping</p>
            </div>
          </UCard>
        </div>
        
        <div v-else>
          <UCard class="mb-6">
            <template #header>
              <h2 class="text-xl font-semibold">Crop Image</h2>
            </template>

            <div class="cropper-container">
              <component 
                :is="Cropper"
                v-if="Cropper"
                ref="cropperRef"
                class="cropper"
                :src="imageUrl"
                :stencil-props="{
                  aspectRatio: currentAspectRatio
                }"
                image-class="cropper-image"
                :canvas="{ width: 1000, height: 1000 }"
                @change="onChange"
              >
                <template #background>
                  <div class="w-full h-full bg-gray-100"></div>
                </template>
              </component>
            </div>
          </UCard>

          <UCard v-if="croppedImage">
            <template #header>
              <h2 class="text-xl font-semibold">Preview</h2>
            </template>

            <div class="preview-container">
              <img :src="croppedImage" alt="Cropped preview" class="max-w-full h-auto rounded-lg" />
            </div>
          </UCard>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
// Import Cropper conditionally for Nuxt compatibility
let Cropper;
// Only import on client-side to avoid SSR issues
if (process.client) {
  Cropper = (await import('vue-advanced-cropper')).Cropper;
  await import('vue-advanced-cropper/dist/style.css');
}

// State variables
const imageUrl = ref('');
const croppedImage = ref(null);
const cropperRef = ref(null);
const zoom = ref(1);
const selectedRatio = ref('1:1');

let lastZoom = 1;

// Computed properties
const hasImage = computed(() => !!imageUrl.value);
const currentAspectRatio = computed(() => {
  const ratio = selectedRatio.value;
  if (ratio === 'free') return undefined;
  const [width, height] = ratio.split(':');
  return Number(width) / Number(height);
});

const aspectRatios = [
  { label: '1:1', value: '1:1' },
  { label: '4:3', value: '4:3' },
  { label: '16:9', value: '16:9' },
  { label: 'Free', value: 'free' }
];

// Methods
function handleFileUpload(event) {
  const file = event.target.files[0];
  if (file && file.type.startsWith('image/')) {
    // Use setTimeout to ensure we have the Cropper component loaded
    setTimeout(() => {
      imageUrl.value = URL.createObjectURL(file);
      resetAdjustments();
    }, 0);
  }
}

function resetImage() {
  imageUrl.value = '';
  croppedImage.value = null;
  resetAdjustments();
}

function resetAdjustments() {
  zoom.value = 1;
  lastZoom = 1;
  if (cropperRef.value) {
    cropperRef.value.refresh();
  }
}

function onChange({ coordinates, canvas }) {
  if (canvas) {
    croppedImage.value = canvas.toDataURL('image/jpeg');
  }
}

function rotateLeft() {
  if (cropperRef.value) {
    cropperRef.value.rotate(-90);
  }
}

function rotateRight() {
  if (cropperRef.value) {
    cropperRef.value.rotate(90);
  }
}

function flipHorizontal() {
  if (cropperRef.value) {
    cropperRef.value.flip(true, false);
  }
}

function flipVertical() {
  if (cropperRef.value) {
    cropperRef.value.flip(false, true);
  }
}

function updateZoom(newZoom) {
  if (cropperRef.value) {
    // Calculate the zoom factor relative to the current zoom
    const factor = newZoom / lastZoom;
    cropperRef.value.zoom(factor);
    lastZoom = newZoom;
  }
}

function downloadImage() {
  if (!croppedImage.value) return;
  
  const link = document.createElement('a');
  link.download = 'cropped-image.jpg';
  link.href = croppedImage.value;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

// Watch for changes in aspect ratio to refresh the cropper
watch(currentAspectRatio, () => {
  if (cropperRef.value) {
    cropperRef.value.refresh();
  }
});

// Force refresh when component is mounted
onMounted(() => {
  // Wait for next tick to ensure the component is fully loaded
  setTimeout(() => {
    if (cropperRef.value) {
      cropperRef.value.refresh();
    }
  }, 100);
});
</script>

<style>
.cropper {
  height: 400px;
  background: #f0f0f0;
  position: relative;
}

.cropper-image {
  transform-origin: center;
}

@media (max-width: 768px) {
  .cropper {
    height: 300px;
  }
}

.preview-container {
  display: flex;
  justify-content: center;
  padding: 1rem;
}

.cropper-placeholder {
  margin-bottom: 1.5rem;
}
</style>