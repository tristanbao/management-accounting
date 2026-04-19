<script setup>
defineProps({
  allocationSteps: { type: Array, required: true },
  traceRows: { type: Array, required: true },
  issueList: { type: Array, required: true },
  money: { type: Function, required: true },
  scenes: { type: Object, required: true },
  activeScene: { type: String, required: true },
  fourCategories: { type: Array, required: true }
})

const emit = defineEmits(['change-scene'])

const sceneTabs = [
  { id: 'customer', label: '客户经营活动', color: '#49dcb1' },
  { id: 'product', label: '产品销售推动', color: '#55c8ff' },
  { id: 'channel', label: '渠道运营支撑', color: '#f7c46a' },
  { id: 'public', label: '公共管理支撑', color: '#e879f9' }
]
</script>

<template>
  <section class="page-grid result-layout">

    <!-- ==================== 场景选择 ==================== -->
    <article class="card panel scene-tabs-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">场景切换</p>
          <h3>选择成本归集场景</h3>
        </div>
      </div>
      <div class="scene-tabs">
        <button
          v-for="tab in sceneTabs"
          :key="tab.id"
          class="scene-tab"
          :class="{ active: activeScene === tab.id }"
          :style="{ '--sc-color': tab.color }"
          @click="emit('change-scene', tab.id)"
        >
          <span>{{ tab.label }}</span>
          <strong>{{ money(scenes[tab.id].amount) }}</strong>
        </button>
      </div>
    </article>

    <!-- ==================== 归集链路流程 ==================== -->
    <article class="card panel flow-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">归集链路</p>
          <h3>成本归集处理明细</h3>
        </div>
        <span class="mini">从原始费用到多维经营对象</span>
      </div>
      <div class="allocation-flow">
        <div class="flow-track">
          <div class="flow-node start">
            <span class="flow-label">原始费用</span>
            <strong>{{ money(scenes[activeScene].amount) }}</strong>
          </div>
          <div class="flow-line"></div>
          <div class="flow-node">
            <span class="flow-label">→ 直接归集</span>
            <strong>{{ money(scenes[activeScene].amount * (scenes[activeScene].methodDetail.direct.ratio / 100)) }}</strong>
            <small>{{ scenes[activeScene].methodDetail.direct.ratio }}% · {{ scenes[activeScene].methodDetail.direct.rule }}</small>
          </div>
          <div class="flow-line"></div>
          <div class="flow-node">
            <span class="flow-label">→ 服务关系</span>
            <strong>{{ money(scenes[activeScene].amount * (scenes[activeScene].methodDetail.service.ratio / 100)) }}</strong>
            <small>{{ scenes[activeScene].methodDetail.service.ratio }}% · {{ scenes[activeScene].methodDetail.service.rule }}</small>
          </div>
          <div class="flow-line"></div>
          <div class="flow-node">
            <span class="flow-label">→ 驱动因子</span>
            <strong>{{ money(scenes[activeScene].amount * (scenes[activeScene].methodDetail.driver.ratio / 100)) }}</strong>
            <small>{{ scenes[activeScene].methodDetail.driver.ratio }}% · {{ scenes[activeScene].methodDetail.driver.rule }}</small>
          </div>
          <div class="flow-line"></div>
          <div class="flow-node">
            <span class="flow-label">→ 公共池</span>
            <strong>{{ money(scenes[activeScene].amount * (scenes[activeScene].methodDetail.pool.ratio / 100)) }}</strong>
            <small>{{ scenes[activeScene].methodDetail.pool.ratio }}% · {{ scenes[activeScene].methodDetail.pool.rule }}</small>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 处理明细表 ==================== -->
    <article class="card panel steps-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">处理明细</p>
          <h3>归集处理步骤</h3>
        </div>
        <span class="mini">沿业务链路逐步落账</span>
      </div>
      <div class="steps-list">
        <div v-for="(step, idx) in allocationSteps" :key="step.step" class="step-item">
          <div class="step-number">{{ String(idx + 1).padStart(2, '0') }}</div>
          <div class="step-content">
            <div class="step-head">
              <strong>{{ step.step }}</strong>
              <span class="step-rule chip">{{ step.rule }}</span>
            </div>
            <p>{{ step.desc }}</p>
          </div>
          <div class="step-amount">
            <strong>{{ money(step.amount) }}</strong>
            <span v-if="idx === 0" class="status-original">原始</span>
            <span v-else class="status-allocated">已归集</span>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 归集追踪表 ==================== -->
    <article class="card panel trace-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">追踪追溯</p>
          <h3>归集结果追踪</h3>
        </div>
        <span class="mini">规则 / 对象 / 台账 / 分析主题</span>
      </div>
      <table>
        <thead>
          <tr>
            <th>处理环节</th>
            <th>归集对象</th>
            <th>规则</th>
            <th>金额</th>
            <th>回写台账</th>
            <th>状态</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in traceRows" :key="row[0]">
            <td><strong>{{ row[0] }}</strong></td>
            <td>{{ row[1] }}</td>
            <td><span class="rule-tag">{{ row[2] }}</span></td>
            <td>{{ money(row[3]) }}</td>
            <td><small>{{ row[4] }}</small></td>
            <td>
              <span class="status-badge" :class="row[5] === '已归集' ? 'status-ok' : row[5] === '已入账' ? 'status-original' : 'status-wait'">
                {{ row[5] }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </article>

    <!-- ==================== 差异预警 ==================== -->
    <article class="card panel issues-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">差异预警</p>
          <h3>归集质量控制规则</h3>
        </div>
        <span class="mini">系统自动校验，归集前发现问题</span>
      </div>
      <div class="issues-list">
        <div v-for="issue in issueList" :key="issue.title" class="issue-item" :class="'severity-' + issue.severity">
          <div class="issue-head">
            <strong>{{ issue.title }}</strong>
            <span class="severity-tag" :class="'sev-' + issue.severity">
              {{ issue.severity === 'high' ? '高优先级' : issue.severity === 'medium' ? '中优先级' : '低优先级' }}
            </span>
          </div>
          <p>{{ issue.desc }}</p>
        </div>
      </div>
    </article>

    <!-- ==================== 归集结果摘要 ==================== -->
    <article class="card panel summary-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">归集摘要</p>
          <h3>当前场景归集结果</h3>
        </div>
      </div>
      <div class="summary-grid">
        <div class="summary-item">
          <span>原始投入</span>
          <strong>{{ money(scenes[activeScene].amount) }}</strong>
        </div>
        <div class="summary-item">
          <span>直接归集</span>
          <strong class="teal">{{ money(scenes[activeScene].amount * scenes[activeScene].methodDetail.direct.ratio / 100) }}</strong>
          <small>{{ scenes[activeScene].methodDetail.direct.ratio }}% → {{ scenes[activeScene].methodDetail.direct.target }}</small>
        </div>
        <div class="summary-item">
          <span>服务关系归集</span>
          <strong class="cyan">{{ money(scenes[activeScene].amount * scenes[activeScene].methodDetail.service.ratio / 100) }}</strong>
          <small>{{ scenes[activeScene].methodDetail.service.ratio }}% → {{ scenes[activeScene].methodDetail.service.target }}</small>
        </div>
        <div class="summary-item">
          <span>驱动因子分摊</span>
          <strong class="gold">{{ money(scenes[activeScene].amount * scenes[activeScene].methodDetail.driver.ratio / 100) }}</strong>
          <small>{{ scenes[activeScene].methodDetail.driver.ratio }}% → {{ scenes[activeScene].methodDetail.driver.target }}</small>
        </div>
        <div class="summary-item">
          <span>公共池分配</span>
          <strong class="purple">{{ money(scenes[activeScene].amount * scenes[activeScene].methodDetail.pool.ratio / 100) }}</strong>
          <small>{{ scenes[activeScene].methodDetail.pool.ratio }}% → {{ scenes[activeScene].methodDetail.pool.target }}</small>
        </div>
        <div class="summary-item highlight">
          <span>ROI</span>
          <strong>{{ scenes[activeScene].roi.roi }}x</strong>
          <small>{{ scenes[activeScene].roi.suggestion }}</small>
        </div>
      </div>
    </article>

  </section>
</template>

<style scoped>
.result-layout {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

/* 场景切换 */
.scene-tabs-panel { grid-column: 1 / -1; }
.scene-tabs { display: flex; gap: 10px; }
.scene-tab {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
  padding: 12px 16px;
  border-radius: 14px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text);
  cursor: pointer;
  text-align: center;
  transition: all .18s ease;
}
.scene-tab:hover { border-color: var(--sc-color); }
.scene-tab.active {
  border-color: var(--sc-color);
  background: color-mix(in srgb, var(--sc-color) 10%, transparent);
}
.scene-tab span { font-size: 13px; color: var(--sc-color); }
.scene-tab strong { font-size: 16px; }

/* 归集链路流程 */
.flow-panel { grid-column: 1 / -1; }
.allocation-flow { overflow-x: auto; }
.flow-track {
  display: flex;
  align-items: center;
  gap: 0;
  min-width: 800px;
}
.flow-node {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 12px 16px;
  border-radius: 12px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  min-width: 120px;
  text-align: center;
}
.flow-node.start { border-color: var(--cyan); background: rgba(85,200,255,.08); }
.flow-label { font-size: 11px; color: var(--muted); }
.flow-node strong { font-size: 15px; margin: 4px 0; }
.flow-node small { font-size: 10px; color: var(--muted); max-width: 100px; }
.flow-line {
  flex: 1;
  height: 2px;
  background: linear-gradient(90deg, var(--line-strong), transparent);
  min-width: 20px;
}

/* 处理明细 */
.steps-list { display: grid; gap: 10px; }
.step-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 12px 14px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.step-number {
  min-width: 32px; height: 32px; line-height: 32px; text-align: center;
  border-radius: 50%; background: rgba(85,200,255,.12); color: var(--cyan);
  font-size: 13px; font-weight: bold;
}
.step-content { flex: 1; }
.step-head { display: flex; align-items: center; gap: 8px; margin-bottom: 4px; }
.step-head strong { font-size: 13px; }
.step-rule { font-size: 10px; }
.step-content p { color: var(--muted); font-size: 12px; margin: 0; }
.step-amount { text-align: right; flex-shrink: 0; }
.step-amount strong { font-size: 15px; }
.step-amount span { display: block; font-size: 10px; padding: 2px 8px; border-radius: 999px; margin-top: 4px; }
.status-original { background: rgba(85,200,255,.15); color: var(--cyan); }
.status-allocated { background: rgba(73,220,177,.15); color: var(--teal); }

/* 标签 */
.rule-tag {
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(85,200,255,.1);
  font-size: 11px;
  font-family: monospace;
}
.status-badge {
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 11px;
}
.status-ok { background: rgba(73,220,177,.15); color: var(--teal); }
.status-original { background: rgba(85,200,255,.15); color: var(--cyan); }
.status-wait { background: rgba(247,196,106,.15); color: var(--gold); }

/* 差异预警 */
.issues-panel { grid-column: 1 / -1; }
.issues-list { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 10px; }
.issue-item {
  padding: 12px 14px;
  border-radius: 12px;
  border-left: 3px solid;
}
.issue-item.severity-high { border-color: #ff6b6b; background: rgba(255,107,107,.06); }
.issue-item.severity-medium { border-color: var(--gold); background: rgba(247,196,106,.06); }
.issue-item.severity-low { border-color: var(--muted); background: rgba(255,255,255,.02); }
.issue-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px; }
.issue-head strong { font-size: 12px; }
.severity-tag { font-size: 10px; padding: 2px 8px; border-radius: 999px; }
.sev-high { background: rgba(255,107,107,.2); color: #ff6b6b; }
.sev-medium { background: rgba(247,196,106,.2); color: var(--gold); }
.sev-low { background: rgba(255,255,255,.06); color: var(--muted); }
.issue-item p { color: var(--muted); font-size: 11px; line-height: 1.6; margin: 0; }

/* 摘要 */
.summary-panel { grid-column: 1 / -1; }
.summary-grid {
  display: grid;
  grid-template-columns: repeat(6, minmax(0, 1fr));
  gap: 12px;
}
.summary-item {
  padding: 14px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  text-align: center;
}
.summary-item.highlight { border-color: var(--cyan); background: rgba(85,200,255,.06); }
.summary-item span { display: block; color: var(--muted); font-size: 11px; margin-bottom: 6px; }
.summary-item strong { display: block; font-size: 18px; }
.summary-item small { display: block; color: var(--muted); font-size: 10px; margin-top: 4px; line-height: 1.4; }
.teal { color: var(--teal); }
.cyan { color: var(--cyan); }
.gold { color: var(--gold); }
.purple { color: #e879f9; }

@media (max-width: 1100px) {
  .issues-list { grid-template-columns: 1fr; }
  .summary-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}
</style>
