<script setup>
import { computed } from 'vue'

const props = defineProps({
  title: { type: String, required: true },
  subtitle: { type: String, required: true },
  contextTitle: { type: String, required: true },
  contextMeta: { type: String, required: true },
  periodLabel: { type: String, required: true },
  periodRange: { type: String, required: true }
})

const emit = defineEmits(['change-period'])
</script>

<template>
  <header class="hero card">
    <div class="hero-left">
      <div class="hero-meta">
        <p class="eyebrow">九江银行 · 多维精益管理 · 成本精细化管理原型</p>
        <h1>{{ title }}</h1>
        <p class="sub">{{ subtitle }}</p>
      </div>
    </div>

    <div class="hero-right">
      <!-- 时间周期选择器 -->
      <div class="period-selector" @click="emit('change-period')">
        <div class="period-badge">
          <span class="period-icon">&#9678;</span>
          <div class="period-info">
            <strong>{{ periodLabel }}</strong>
            <small>{{ periodRange }}</small>
          </div>
          <span class="period-arrow">&#9662;</span>
        </div>
        <div class="period-tip">点击切换周期</div>
      </div>

      <!-- 当前归集方法 -->
      <div class="badge">
        <span>当前归集方法</span>
        <strong>{{ contextTitle }}</strong>
        <small>{{ contextMeta }}</small>
      </div>
    </div>
  </header>
</template>

<style scoped>
.hero {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 24px;
  padding: 24px 28px;
}

.hero-left { flex: 1; min-width: 0; }

.hero-meta .eyebrow {
  margin: 0 0 6px;
  font-size: 11px;
  color: var(--muted);
  letter-spacing: .05em;
}

.hero-meta h1 { margin: 0 0 8px; font-size: 22px; }
.hero-meta .sub { margin: 0; font-size: 13px; color: var(--muted); line-height: 1.6; max-width: 600px; }

.hero-right {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 12px;
  flex-shrink: 0;
}

/* 时间周期选择器 */
.period-selector {
  cursor: pointer;
  user-select: none;
}

.period-badge {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  border-radius: 12px;
  background: rgba(73,220,177,.1);
  border: 1px solid rgba(73,220,177,.3);
  transition: all .18s ease;
}

.period-selector:hover .period-badge {
  background: rgba(73,220,177,.18);
  border-color: rgba(73,220,177,.5);
}

.period-icon {
  font-size: 10px;
  color: var(--teal);
  animation: blink 2s ease-in-out infinite;
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: .3; }
}

.period-info { text-align: left; }
.period-info strong { display: block; font-size: 14px; color: var(--teal); }
.period-info small { display: block; font-size: 11px; color: var(--muted); margin-top: 2px; }
.period-arrow { font-size: 10px; color: var(--teal); }

.period-tip {
  text-align: center;
  font-size: 10px;
  color: var(--muted);
  margin-top: 4px;
  opacity: .6;
}

.badge {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 2px;
  padding: 10px 14px;
  border-radius: 12px;
  background: rgba(85,200,255,.08);
  border: 1px solid rgba(85,200,255,.2);
  text-align: right;
}
.badge span { font-size: 10px; color: var(--muted); }
.badge strong { font-size: 13px; color: var(--cyan); }
.badge small { font-size: 11px; color: var(--muted); }
</style>
