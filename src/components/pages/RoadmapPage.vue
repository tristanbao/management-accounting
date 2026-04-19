<script setup>
defineProps({
  roadmap: { type: Array, required: true },
  systemUpgrades: { type: Array, required: true }
})
</script>

<template>
  <section class="page-grid two-col">
    <article class="card panel">
      <div class="title-row">
        <h3>分阶段落地路线图</h3>
        <span class="mini">先规则、后试点、再推广</span>
      </div>
      <div class="stack-list roadmap-list">
        <div v-for="item in roadmap" :key="item.phase" class="stack-item phase-card">
          <div class="phase-head">
            <strong>{{ item.phase }}</strong>
            <span class="chip">{{ item.goal }}</span>
          </div>
          <ul class="bullet-list">
            <li v-for="task in item.tasks" :key="task">{{ task }}</li>
          </ul>
        </div>
      </div>
    </article>

    <article class="card panel">
      <div class="title-row">
        <h3>外围系统改造清单</h3>
        <span class="mini">页面、字段、接口、台账</span>
      </div>
      <div class="stack-list compact">
        <div v-for="item in systemUpgrades" :key="item.system" class="stack-item blocky">
          <div>
            <strong>{{ item.system }}</strong>
            <p>{{ item.change }}</p>
          </div>
          <small>{{ item.level }}</small>
        </div>
      </div>
    </article>
  </section>
</template>
