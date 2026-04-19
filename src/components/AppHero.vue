<script setup>
defineProps({
  title: { type: String, required: true },
  subtitle: { type: String, required: true },
  contextTitle: { type: String, required: true },
  contextMeta: { type: String, required: true }
})
</script>

<template>
  <header class="hero card">
    <div>
      <p class="eyebrow">九江银行 · 多维精益管理 · 成本精细化管理原型</p>
      <h1>{{ title }}</h1>
      <p class="sub">{{ subtitle }}</p>
    </div>

    <div class="badge">
      <span>当前归集方法</span>
      <strong>{{ contextTitle }}</strong>
      <small>{{ contextMeta }}</small>
    </div>
  </header>
</template>
