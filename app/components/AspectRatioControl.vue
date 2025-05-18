<template>
  <div>
    <label class="block text-sm font-medium mb-2">Aspect Ratio</label>
    <UButtonGroup class="w-full" block>
      <UButton
        v-for="(ratio, index) in ratios"
        :key="index"
        :color="modelValue === ratio.value ? 'primary' : 'gray'"
        variant="soft"
        @click="$emit('update:modelValue', ratio.value)"
      >
        {{ ratio.label }}
      </UButton>
    </UButtonGroup>
  </div>
</template>

<script setup>
defineProps({
  modelValue: {
    type: String,
    default: '1:1'
  },
  ratios: {
    type: Array,
    default: () => [
      { label: '1:1', value: '1:1' },
      { label: '4:3', value: '4:3' },
      { label: '16:9', value: '16:9' },
      { label: 'Free', value: 'free' }
    ]
  }
});

defineEmits(['update:modelValue']);
</script>