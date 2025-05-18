<template>
  <div>
    <UTabs
      color="success"
      variant="pill"
      class="mb-8 w-full"
      v-model="activeTab"
      :items="tabs"
    >
      <template #advanced>
        <CropperPage />
      </template>

      <template #custom>
        <TabbedCropper />
      </template>
    </UTabs>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import type { TabsItem } from '@nuxt/ui'
import CropperPage from './CropperPage.vue'
import TabbedCropper from './TabbedCropper.vue'

// Default active tab
const activeTab = ref<'advanced' | 'custom'>('advanced')

const tabs: TabsItem[] = [
  {
    label: 'Advanced Size Cropper',
    slot: 'advanced',
    value: 'advanced'
  },
  {
    label: 'Custom Size Cropper',
    slot: 'custom',
    value: 'custom'
  }
]
</script>
