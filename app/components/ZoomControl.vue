<template>
  <div>
    <label class="block text-sm font-medium mb-2">Zoom: {{ modelValue.toFixed(1) }}x</label>
    <URange 
      :model-value="modelValue"
      :min="1" 
      :max="3" 
      :step="0.1"
      class="w-full"
      @update:model-value="$emit('update:modelValue', $event)"
    />
  </div>
</template>

<script setup>
defineProps({
  modelValue: {
    type: Number,
    default: 1
  }
});

defineEmits(['update:modelValue']);
</script>