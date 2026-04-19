<script setup>
defineProps({
  productRows: { type: Array, required: true },
  roiSummary: { type: Array, required: true },
  scenes: { type: Object, required: true },
  activeScene: { type: String, required: true },
  activePeriod: { type: String, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-scene'])

const sceneTabs = [
  { id: 'customer', label: '客户经营活动', color: '#49dcb1' },
  { id: 'product', label: '产品销售推动', color: '#55c8ff' },
  { id: 'channel', label: '渠道运营支撑', color: '#f7c46a' },
  { id: 'public', label: '公共管理支撑', color: '#e879f9' }
]

const dimensionTabs = [
  { id: 'customer', label: '客户视角', icon: '👥' },
  { id: 'product', label: '产品视角', icon: '💰' },
  { id: 'channel', label: '渠道视角', icon: '📱' },
  { id: 'management', label: '管理视角', icon: '🏦' }
]

const activeDimension = 'customer'
</script>

<template>
  <section class="page-grid roi-layout">

    <!-- ==================== 场景选择 ==================== -->
    <article class="card panel scene-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">场景切换</p>
          <h3>选择分析场景</h3>
        </div>
        <div class="period-badge">
          <span>&#9678;</span>
          <span>{{ activePeriod }}</span>
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
          <small>投入</small>
        </button>
      </div>
    </article>

    <!-- ==================== 多维分析视角 ==================== -->
    <article class="card panel dimensions-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">多维分析</p>
          <h3>从四个视角回看投入产出</h3>
        </div>
        <span class="mini">点击切换分析维度</span>
      </div>
      <div class="dimension-tabs">
        <button
          v-for="dim in dimensionTabs"
          :key="dim.id"
          class="dimension-tab"
          :class="{ active: activeDimension === dim.id }"
        >
          <span class="dim-icon">{{ dim.icon }}</span>
          <strong>{{ dim.label }}</strong>
        </button>
      </div>
      <div class="dimension-cards">
        <div v-for="item in roiSummary" :key="item.title" class="dimension-card">
          <div class="dim-head">
            <strong>{{ item.title }}</strong>
            <span class="dim-type-tag">{{ item.type }}</span>
          </div>
          <p>{{ item.desc }}</p>
        </div>
      </div>
    </article>

    <!-- ==================== ROI总览 ==================== -->
    <article class="card panel roi-overview-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">当前场景</p>
          <h3>{{ scenes[activeScene].label }}投入产出</h3>
        </div>
        <span class="mini">{{ scenes[activeScene].method }}</span>
      </div>
      <div class="roi-cards">
        <div class="roi-card primary">
          <span>收入贡献</span>
          <strong>{{ money(scenes[activeScene].roi.revenue || 0) }}</strong>
          <p>{{ scenes[activeScene].roi.revenue > 0 ? '已匹配收入' : '间接效益待评估' }}</p>
        </div>
        <div class="roi-card">
          <span>成本投入</span>
          <strong>{{ money(scenes[activeScene].amount) }}</strong>
          <p>归集后总成本</p>
        </div>
        <div class="roi-card">
          <span>ROI</span>
          <strong :class="scenes[activeScene].roi.roi >= 1.5 ? 'teal' : scenes[activeScene].roi.roi >= 1 ? 'gold' : 'red'">
            {{ scenes[activeScene].roi.roi > 0 ? scenes[activeScene].roi.roi + 'x' : '—' }}
          </strong>
          <p>{{ scenes[activeScene].roi.suggestion }}</p>
        </div>
      </div>
    </article>

    <!-- ==================== 投入产出明细 ==================== -->
    <article class="card panel product-table-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">经营分析</p>
          <h3>多对象投入产出明细</h3>
        </div>
        <span class="mini">支持客户、产品、渠道、客群维度</span>
      </div>
      <table>
        <thead>
          <tr>
            <th>对象名称</th>
            <th>类型</th>
            <th>客户数/服务量</th>
            <th>收入</th>
            <th>归集成本</th>
            <th>ROI</th>
            <th>状态</th>
            <th>建议动作</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="r in productRows" :key="r.name">
            <td><strong>{{ r.name }}</strong></td>
            <td><span class="type-tag">{{ r.type }}</span></td>
            <td>{{ r.clients.toLocaleString() }}</td>
            <td>{{ money(r.revenue) }}</td>
            <td>{{ money(r.cost) }}</td>
            <td>
              <span class="roi-badge" :class="r.roi >= 2 ? 'roi-high' : r.roi >= 1.5 ? 'roi-mid' : 'roi-low'">
                {{ r.roi }}x
              </span>
            </td>
            <td><span class="status-tag" :class="r.status.includes('优') ? 'status-ok' : r.status.includes('待') ? 'status-warn' : 'status-info'">{{ r.status }}</span></td>
            <td><small>{{ r.suggestion }}</small></td>
          </tr>
        </tbody>
      </table>
    </article>

    <!-- ==================== 经营优化建议 ==================== -->
    <article class="card panel optimization-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">决策支持</p>
          <h3>经营优化建议</h3>
        </div>
        <span class="mini">基于归集结果与ROI分析</span>
      </div>
      <div class="optimization-list">
        <div class="opt-item">
          <div class="opt-head">
            <span class="opt-icon">📈</span>
            <strong>加大高ROI投入</strong>
            <span class="opt-tag positive">优先执行</span>
          </div>
          <p>白鹭信用卡新客礼遇（ROI 1.96x）和高净值客群经营（ROI 2.12x）表现优异，建议加大线上获客投入规模，提升获客效率。</p>
          <div class="opt-detail">
            <span>预期增量投入：¥500,000</span>
            <span>预期增量收入：¥980,000</span>
            <span>预期增量ROI：2.0x+</span>
          </div>
        </div>
        <div class="opt-item">
          <div class="opt-head">
            <span class="opt-icon">⚙️</span>
            <strong>优化渠道配置</strong>
            <span class="opt-tag warning">重点关注</span>
          </div>
          <p>普惠经营贷通过企业网银渠道ROI偏低（1.67x），建议评估渠道效率，优化线上线下渠道资源配置，提升渠道转化率。</p>
          <div class="opt-detail">
            <span>当前渠道ROI：1.67x</span>
            <span>目标渠道ROI：2.0x+</span>
            <span>建议措施：渠道组合优化</span>
          </div>
        </div>
        <div class="opt-item">
          <div class="opt-head">
            <span class="opt-icon">🎯</span>
            <strong>精准客群经营</strong>
            <span class="opt-tag neutral">持续优化</span>
          </div>
          <p>稳利盈6M理财（财富产品，ROI 1.79x）维持稳定，建议持续深耕AUM≥600万高净值客群，稳步提升客户AUM和综合贡献度。</p>
          <div class="opt-detail">
            <span>目标客群：AUM≥600万</span>
            <span>重点动作：沙龙+权益+专属服务</span>
            <span>预期效果：客户AUM年增长15%</span>
          </div>
        </div>
      </div>
    </article>

  </section>
</template>

<style scoped>
.roi-layout {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.title-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 16px;
}

.period-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border-radius: 999px;
  background: rgba(73,220,177,.1);
  border: 1px solid rgba(73,220,177,.3);
  font-size: 12px;
  color: var(--teal);
  flex-shrink: 0;
}
.period-badge span:first-child {
  font-size: 8px;
  animation: blink 2s ease-in-out infinite;
}
@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: .3; }
}

/* 场景选择 */
.scene-panel { grid-column: 1 / -1; }
.scene-tabs { display: flex; gap: 10px; }
.scene-tab {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 12px;
  border-radius: 14px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text);
  cursor: pointer;
  transition: all .18s ease;
}
.scene-tab:hover { border-color: var(--sc-color); }
.scene-tab.active {
  border-color: var(--sc-color);
  background: color-mix(in srgb, var(--sc-color) 10%, transparent);
}
.scene-tab span { font-size: 13px; color: var(--sc-color); }
.scene-tab strong { font-size: 18px; }
.scene-tab small { font-size: 11px; color: var(--muted); }

/* 多维分析视角 */
.dimensions-panel { grid-column: 1 / -1; }
.dimension-tabs { display: flex; gap: 10px; margin-bottom: 14px; }
.dimension-tab {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  border-radius: 12px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text);
  cursor: pointer;
  font-size: 13px;
  transition: all .18s ease;
}
.dimension-tab:hover { border-color: var(--line-strong); }
.dimension-tab.active {
  border-color: var(--cyan);
  background: rgba(85,200,255,.1);
}
.dim-icon { font-size: 18px; }

.dimension-cards {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}
.dimension-card {
  padding: 14px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.dim-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.dim-head strong { font-size: 13px; }
.dim-type-tag {
  padding: 3px 10px;
  border-radius: 999px;
  background: rgba(85,200,255,.1);
  color: var(--cyan);
  font-size: 11px;
}
.dimension-card p { color: var(--muted); font-size: 12px; line-height: 1.7; margin: 0; }

/* ROI总览 */
.roi-overview-panel { grid-column: 1 / -1; }
.roi-cards {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 14px;
}
.roi-card {
  padding: 18px;
  border-radius: 16px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  text-align: center;
}
.roi-card.primary {
  border-color: var(--cyan);
  background: rgba(85,200,255,.06);
}
.roi-card span { display: block; color: var(--muted); font-size: 12px; margin-bottom: 8px; }
.roi-card strong { display: block; font-size: 28px; margin-bottom: 6px; }
.roi-card p { color: var(--muted); font-size: 12px; margin: 0; }
.teal { color: var(--teal); }
.gold { color: var(--gold); }
.red { color: #ff6b6b; }

/* 投入产出明细 */
.product-table-panel { grid-column: 1 / -1; }

.type-tag {
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(85,200,255,.1);
  font-size: 11px;
}

.roi-badge {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 12px;
  font-weight: bold;
}
.roi-high { background: rgba(73,220,177,.2); color: var(--teal); }
.roi-mid { background: rgba(247,196,106,.2); color: var(--gold); }
.roi-low { background: rgba(255,107,107,.15); color: #ff6b6b; }

.status-tag {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 11px;
}
.status-ok { background: rgba(73,220,177,.15); color: var(--teal); }
.status-warn { background: rgba(247,196,106,.15); color: var(--gold); }
.status-info { background: rgba(85,200,255,.12); color: var(--cyan); }

/* 优化建议 */
.optimization-panel { grid-column: 1 / -1; }
.optimization-list { display: grid; gap: 12px; }
.opt-item {
  padding: 16px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.opt-head { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
.opt-icon { font-size: 18px; }
.opt-head strong { font-size: 14px; flex: 1; }
.opt-tag {
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 11px;
}
.opt-tag.positive { background: rgba(73,220,177,.15); color: var(--teal); }
.opt-tag.warning { background: rgba(247,196,106,.15); color: var(--gold); }
.opt-tag.neutral { background: rgba(85,200,255,.12); color: var(--cyan); }
.opt-item p { color: var(--muted); font-size: 12px; line-height: 1.7; margin: 0 0 10px; }
.opt-detail {
  display: flex;
  gap: 16px;
  padding-top: 10px;
  border-top: 1px solid var(--line);
}
.opt-detail span { font-size: 11px; color: var(--muted); }

@media (max-width: 1100px) {
  .dimension-cards { grid-template-columns: 1fr; }
  .roi-cards { grid-template-columns: 1fr; }
}
</style>
