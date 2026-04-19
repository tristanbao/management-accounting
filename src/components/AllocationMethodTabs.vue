<script setup>
defineProps({
  methods: { type: Array, required: true },
  activeMethod: { type: String, required: true }
})

const emit = defineEmits(['change'])
</script>

<template>
  <div class="method-tabs">
    <button
      v-for="item in methods"
      :key="item.id"
      class="method-tab"
      :class="{ active: item.id === activeMethod }"
      @click="emit('change', item.id)"
    >
      <strong>{{ item.label }}</strong>
      <small>{{ item.scenario }}</small>
    </button>
  </div>
</template>
