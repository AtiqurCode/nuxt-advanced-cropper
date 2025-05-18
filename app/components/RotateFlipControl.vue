<template>
  <div>
    <label class="block text-sm font-medium mb-2">Rotate & Flip</label>
    <div class="flex space-x-2">
      <UButton 
        icon="i-lucide-rotate-ccw" 
        color="gray" 
        variant="soft"
        @click="$emit('rotate-left')"
        class="flex-1"
      >
        Left
      </UButton>
      <UButton 
        icon="i-lucide-rotate-cw"
        color="gray" 
        variant="soft"
        @click="$emit('rotate-right')"
        class="flex-1"
      >
        Right
      </UButton>
      <UButton
        icon="i-lucide-flip-horizontal"
        color="gray"
        variant="soft"
        @click="$emit('flip-horizontal')"
        class="flex-1"
      >
        Flip
      </UButton>
      <UButton
        icon="i-lucide-flip-vertical"
        color="gray"
        variant="soft"
        @click="$emit('flip-vertical')"
        class="flex-1"
      >
        Flip
      </UButton>
    </div>
  </div>
</template>

<script setup>
defineEmits(['rotate-left', 'rotate-right', 'flip-horizontal', 'flip-vertical']);
</script>