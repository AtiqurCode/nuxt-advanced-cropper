<template>
  <div>
    <header class="mb-8 text-center">
      <h1 class="text-3xl font-bold mb-2">Custom Image Cropper</h1>
      <p class="text-gray-500">Crop your images with different options</p>
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
            <h2 class="text-xl font-semibold">Crop Settings</h2>
          </template>
          
          <UTabs color="primary" variant="pill" v-model="activeTabIndex" :items="tabs" class="w-full" />
          
          <div class="mt-4">
            <!-- Custom Size Tab Content -->
            <div v-if="activeTabIndex === 'custom'" class="space-y-4">
              <div class="grid grid-cols-2 gap-4">
                <div>
                  <label class="block text-sm font-medium mb-2">Width (px)</label>
                  <UInput
                    v-model="customWidth"
                    type="number"
                    min="1"
                    @update:model-value="updateCustomSize"
                  />
                </div>
                <div>
                  <label class="block text-sm font-medium mb-2">Height (px)</label>
                  <UInput
                    v-model="customHeight"
                    type="number"
                    min="1"
                    @update:model-value="updateCustomSize"
                  />
                </div>
              </div>
              <UButton
                color="gray"
                variant="soft"
                block
                @click="lockAspectRatio = !lockAspectRatio"
              >
                <UIcon
                  :name="lockAspectRatio ? 'i-heroicons-lock-closed' : 'i-heroicons-lock-open'"
                  class="mr-1"
                />
                {{ lockAspectRatio ? 'Unlock Aspect Ratio' : 'Lock Aspect Ratio' }}
              </UButton>
            </div>

            <!-- Preset Sizes Tab Content -->
            <div v-if="activeTabIndex === 'preset'" class="space-y-4">
              <div v-for="(size, index) in presetSizes" :key="index">
                <UButton
                  block
                  :color="selectedPreset === size ? 'primary' : 'gray'"
                  variant="soft"
                  class="mb-2"
                  @click="selectPreset(size)"
                >
                  {{ size.label }}
                </UButton>
              </div>
            </div>
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
                :stencil-props="stencilProps"
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
if (process.client) {
  Cropper = (await import('vue-advanced-cropper')).Cropper;
  await import('vue-advanced-cropper/dist/style.css');
}

// State variables
const imageUrl = ref('');
const croppedImage = ref(null);
const cropperRef = ref(null);
const activeTabIndex = ref(0);
const customWidth = ref(800);
const customHeight = ref(600);
const lockAspectRatio = ref(true);
const selectedPreset = ref(null);

// Tabs configuration

const tabs = [
  {
    label: 'Custom Size',
    slot: 'custom',
    value: 'custom'
  },
  {
    label: 'Preset Sizes',
    slot: 'preset',
    value: 'preset'
  }
]

// Preset sizes
const presetSizes = [
  { label: 'Profile Picture (400x400)', width: 400, height: 400 },
  { label: 'Cover Photo (1200x630)', width: 1200, height: 630 },
  { label: 'Instagram Post (1080x1080)', width: 1080, height: 1080 },
  { label: 'Twitter Header (1500x500)', width: 1500, height: 500 }
];

// Computed properties
const hasImage = computed(() => !!imageUrl.value);

const stencilProps = computed(() => {
  if (activeTabIndex.value === 'custom') {
    return {
      aspectRatio: lockAspectRatio.value ? customWidth.value / customHeight.value : undefined,
      width: customWidth.value,
      height: customHeight.value
    };
  } else {
    const preset = selectedPreset.value;
    return preset ? {
      aspectRatio: preset.width / preset.height,
      width: preset.width,
      height: preset.height
    } : {
      aspectRatio: 1,
      width: 400,
      height: 400
    };
  }
});

// Methods
function handleFileUpload(event) {
  const file = event.target.files[0];
  if (file && file.type.startsWith('image/')) {
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
  if (cropperRef.value) {
    cropperRef.value.refresh();
  }
}

function onChange({ coordinates, canvas }) {
  if (canvas) {
    croppedImage.value = canvas.toDataURL('image/jpeg');
  }
}

function updateCustomSize() {
  if (cropperRef.value) {
    cropperRef.value.refresh();
  }
}

function selectPreset(size) {
  selectedPreset.value = size;
  if (cropperRef.value) {
    cropperRef.value.refresh();
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

// Watch for changes and refresh cropper
watch([activeTabIndex, stencilProps], () => {
  if (cropperRef.value) {
    cropperRef.value.refresh();
  }
});

// Force refresh when component is mounted
onMounted(() => {
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