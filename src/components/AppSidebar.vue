<script setup>
defineProps({
  navItems: { type: Array, required: true },
  activePage: { type: String, required: true }
})

const emit = defineEmits(['navigate'])
</script>

<template>
  <aside class="sidebar card">
    <div class="brand">
      <p class="eyebrow">JIUJIANG BANK</p>
      <h2>成本归集改造</h2>
      <p class="brand-sub">三层递进 · 四分法 · 流程嵌入式改造</p>
    </div>

    <button
      v-for="item in navItems"
      :key="item.id"
      class="nav-btn"
      :class="{ active: activePage === item.id }"
      @click="emit('navigate', item.id)"
    >
      <span>{{ item.label }}</span>
      <small>{{ item.desc }}</small>
    </button>

    <div class="side-tip">
      <strong>设计原则</strong>
      <p>不是做一个总控分摊页，而是把四分法归集要求分别嵌入：费用申请、客户经理工时、驱动因子台账、公共成本池管理四大业务场景。</p>
    </div>

    <div class="method-legend">
      <div class="legend-item">
        <span class="dot" style="background:#49dcb1"></span>
        <small>第一分法：直连对象</small>
      </div>
      <div class="legend-item">
        <span class="dot" style="background:#55c8ff"></span>
        <small>第二分法：服务关系</small>
      </div>
      <div class="legend-item">
        <span class="dot" style="background:#f7c46a"></span>
        <small>第三分法：驱动因子</small>
      </div>
      <div class="legend-item">
        <span class="dot" style="background:#e879f9"></span>
        <small>第四分法：公共成本池</small>
      </div>
    </div>
  </aside>
</template>

<style scoped>
.brand-sub {
  font-size: 11px;
  color: var(--cyan);
  margin: 4px 0 0;
}

.method-legend {
  margin-top: 6px;
  padding: 12px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 6px;
}
.legend-item:last-child { margin-bottom: 0; }

.dot {
  width: 8px; height: 8px; border-radius: 50%;
  flex-shrink: 0;
}
.legend-item small { color: var(--muted); font-size: 11px; }
</style>
