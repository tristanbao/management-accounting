<script setup>
import { computed, reactive, ref } from 'vue'

const props = defineProps({
  scenes: { type: Object, required: true },
  activeScene: { type: String, required: true },
  activePeriod: { type: String, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-scene'])

// ============================================================
// 维度定义：每个维度对应一种分析视角
// ============================================================
const dimensions = [
  {
    id: 'customer', label: '客户视角', icon: '👥',
    focus: '客群经营资源配置效率',
    metrics: ['客户数', 'AUM', '户均AUM', '获客成本', '客户留存率'],
    compareLabel: '客群之间横向对比'
  },
  {
    id: 'product', label: '产品视角', icon: '💰',
    focus: '产品投入产出与盈利结构',
    metrics: ['产品收入', '归集成本', '成本收入比', '边际贡献', '生命周期阶段'],
    compareLabel: '产品线之间横向对比'
  },
  {
    id: 'channel', label: '渠道视角', icon: '📱',
    focus: '渠道效能与资源配置',
    metrics: ['交易量', '交易金额', '渠道成本', '单笔成本', '渠道转化率'],
    compareLabel: '渠道之间横向对比'
  },
  {
    id: 'management', label: '管理视角', icon: '🏦',
    focus: '条线盈利评价与公共资源分配',
    metrics: ['条线收入', '条线成本', '条线EVA', '费用收入比', '分摊合理性'],
    compareLabel: '条线之间横向对比'
  }
]

const activeDimension = ref('customer')

// ============================================================
// 场景配置：定义每个场景在各维度下的分析数据
// ============================================================
const sceneAnalysisData = computed(() => {
  const s = props.scenes[props.activeScene]
  return {
    customer: {
      // 客户视角分析数据
      kpis: [
        { label: '目标客群', value: s.customer || '—', sub: '重点服务对象' },
        { label: '客群总人数', value: s.clients || '—', sub: 'AUM≥600万' },
        { label: '本期获客数', value: s.newClients || 38, sub: '环比+12%' },
        { label: '户均AUM', value: s.avgAum || '¥820万', sub: '行内均值¥480万' },
        { label: '获客成本(CAC)', value: s.cac || '¥4,860', sub: '行业均值¥6,200' },
        { label: '客群ROI', value: s.roi.roi > 0 ? s.roi.roi + 'x' : '间接', sub: '高于行内均值1.65x' }
      ],
      // 客户分群对比（仅客户经营场景有完整数据）
      segments: props.activeScene === 'customer' ? [
        { name: 'A03高净值客群', clients: 328, aum: '26.9亿', revenue: 357000, cost: 186000, roi: 1.92, status: '优', trend: 'up' },
        { name: 'A02普通财富客群', clients: 892, aum: '35.7亿', revenue: 412000, cost: 268000, roi: 1.54, status: '良', trend: 'stable' },
        { name: 'A01基础客群', clients: 4820, aum: '48.2亿', revenue: 386000, cost: 312000, roi: 1.24, status: '待优化', trend: 'down' }
      ] : [
        { name: '主要受益客群', clients: s.clients || 328, aum: s.avgAum || '¥820万', revenue: s.roi.revenue || 0, cost: s.amount, roi: s.roi.roi || 0, status: s.roi.roi >= 1.5 ? '优' : s.roi.roi >= 1 ? '良' : '待优化', trend: 'up' }
      ],
      waterfall: s.methodDetail ? [
        { step: '直接归集', amount: s.methodDetail.direct?.amount || 0, ratio: s.methodDetail.direct?.ratio || 0, desc: '可直连客群的费用', color: '#49dcb1' },
        { step: '服务关系归集', amount: s.methodDetail.service?.amount || 0, ratio: s.methodDetail.service?.ratio || 0, desc: '管户经理工时分摊', color: '#55c8ff' },
        { step: '驱动因子分摊', amount: s.methodDetail.driver?.amount || 0, ratio: s.methodDetail.driver?.ratio || 0, desc: '按活跃度等因子分配', color: '#f7c46a' },
        { step: '公共池分摊', amount: s.methodDetail.pool?.amount || 0, ratio: s.methodDetail.pool?.ratio || 0, desc: '管理支撑成本分配', color: '#e879f9' }
      ] : []
    },
    product: {
      // 产品视角分析数据
      kpis: [
        { label: '主要产品', value: s.product?.split('/')[0]?.trim() || '—', sub: '重点推动产品' },
        { label: '本期收入', value: props.money(s.roi?.revenue || 0), sub: '归集成本收益匹配' },
        { label: '成本收入比', value: s.costIncomeRatio || '64.9%', sub: '参考值<50%' },
        { label: '边际贡献', value: s.margin || '¥28.6万', sub: '收入-直接成本' },
        { label: '市场份额', value: s.marketShare || '8.2%', sub: '区域市场' },
        { label: '产品ROI', value: s.roi.roi > 0 ? s.roi.roi + 'x' : '—', sub: '投入产出比' }
      ],
      // 产品对比
      products: [
        { name: s.product?.split('/')[0]?.trim() || '主要产品', revenue: s.roi?.revenue || 0, cost: s.amount, roi: s.roi.roi || 0, rank: 1 },
        { name: '竞品A（他行同类）', revenue: 720000, cost: 486000, roi: 1.48, rank: 2 },
        { name: '竞品B（行内同类）', revenue: 380000, cost: 298000, roi: 1.28, rank: 3 }
      ],
      waterfall: s.methodDetail ? [
        { step: '产品直归', amount: s.methodDetail.direct?.amount || 0, ratio: s.methodDetail.direct?.ratio || 0, desc: '可直连产品的推广费', color: '#49dcb1' },
        { step: '服务关系归集', amount: s.methodDetail.service?.amount || 0, ratio: s.methodDetail.service?.ratio || 0, desc: '产品经理支持工时', color: '#55c8ff' },
        { step: '驱动因子分摊', amount: s.methodDetail.driver?.amount || 0, ratio: s.methodDetail.driver?.ratio || 0, desc: '按转化客户数分摊', color: '#f7c46a' },
        { step: '公共池分摊', amount: s.methodDetail.pool?.amount || 0, ratio: s.methodDetail.pool?.ratio || 0, desc: '产品线管理支撑', color: '#e879f9' }
      ] : []
    },
    channel: {
      // 渠道视角分析数据
      kpis: [
        { label: '主渠道', value: s.channel?.split('/')[0]?.trim() || '—', sub: '核心获客渠道' },
        { label: '交易笔数', value: s.transactionCount || '128万笔', sub: '本期全渠道' },
        { label: '交易金额', value: s.transactionAmount || '¥42.6亿', sub: '本期全渠道' },
        { label: '渠道成本', value: props.money(s.amount), sub: '归集总成本' },
        { label: '单笔成本', value: s.costPerTxn || '¥2.67', sub: '成本÷交易笔数' },
        { label: '渠道ROI', value: s.roi.roi > 0 ? s.roi.roi + 'x' : '—', sub: '数字渠道领先' }
      ],
      channels: [
        { name: '手机银行', txn: 537600, amount: '18.3亿', cost: 143640, roi: 3.2, status: '最优', trend: 'up', fill: 85 },
        { name: '网上银行', txn: 358400, amount: '12.1亿', cost: 95600, roi: 2.6, status: '良好', trend: 'up', fill: 65 },
        { name: '营业网点', txn: 294400, amount: '9.8亿', cost: 102600, roi: 1.8, status: '一般', trend: 'stable', fill: 45 },
        { name: '客服中心', txn: 89600, amount: '2.4亿', cost: 0, roi: 1.4, status: '待优化', trend: 'down', fill: 25 }
      ],
      waterfall: s.methodDetail ? [
        { step: '渠道专属直归', amount: s.methodDetail.direct?.amount || 0, ratio: s.methodDetail.direct?.ratio || 0, desc: '网点专属设备等', color: '#49dcb1' },
        { step: '服务关系归集', amount: s.methodDetail.service?.amount || 0, ratio: s.methodDetail.service?.ratio || 0, desc: '渠道服务工时分摊', color: '#55c8ff' },
        { step: '交易量驱动分摊', amount: s.methodDetail.driver?.amount || 0, ratio: s.methodDetail.driver?.ratio || 0, desc: '按交易量占比分配', color: '#f7c46a' },
        { step: '公共池分摊', amount: s.methodDetail.pool?.amount || 0, ratio: s.methodDetail.pool?.ratio || 0, desc: '渠道公共支撑', color: '#e879f9' }
      ] : []
    },
    management: {
      // 管理视角分析数据
      kpis: [
        { label: '分摊方法', value: s.method?.split('（')[0]?.trim() || '—', sub: '适用分摊法' },
        { label: '条线收入', value: props.money(s.roi?.revenue || 0), sub: '条线贡献收入' },
        { label: '条线成本', value: props.money(s.amount), sub: '归集后总成本' },
        { label: '条线EVA', value: s.eva || '¥8.4万', sub: '收入-成本-资本成本' },
        { label: '费用收入比', value: s.costIncomeRatio || '64.9%', sub: '参考值<40%' },
        { label: '分摊合理性', value: s.allocationQuality || '良好', sub: '公共池分摊合理性评估' }
      ],
      lines: [
        { name: '零售金融部', revenue: 1200000, cost: s.amount || 528000, eva: 84000, roi: s.roi.roi || 1.54, ratio: 45, status: '盈利', trend: 'up' },
        { name: '对公业务部', revenue: 980000, cost: 620000, eva: 52000, roi: 1.58, ratio: 32, status: '盈利', trend: 'stable' },
        { name: '普惠金融部', revenue: 420000, cost: 385000, eva: -12000, roi: 1.09, ratio: 23, status: '亏损边缘', trend: 'down' }
      ],
      waterfall: s.methodDetail ? [
        { step: '公共成本直归', amount: s.methodDetail.direct?.amount || 0, ratio: s.methodDetail.direct?.ratio || 0, desc: '可直接归属的公共费', color: '#49dcb1' },
        { step: '管理服务归集', amount: s.methodDetail.service?.amount || 0, ratio: s.methodDetail.service?.ratio || 0, desc: '各条线管理支撑工时', color: '#55c8ff' },
        { step: '资源驱动分摊', amount: s.methodDetail.driver?.amount || 0, ratio: s.methodDetail.driver?.ratio || 0, desc: '按面积/人数/收入', color: '#f7c46a' },
        { step: '逐级池分配', amount: s.methodDetail.pool?.amount || 0, ratio: s.methodDetail.pool?.ratio || 0, desc: '总行→分行→条线→网点', color: '#e879f9' }
      ] : []
    }
  }
})

// ============================================================
// 周期趋势数据（基于App.vue中的periodData逻辑）
// ============================================================
const trendPeriods = ['2024FY', '2025H1', '2025Q3', '2025Q4', '2026Q1']

const periodTrend = computed(() => {
  const sceneAmounts = {
    customer: [1102000, 468000, 142000, 158000, 186000],
    product: [1628000, 718000, 396000, 462000, 528000],
    channel: [2268000, 986000, 276000, 298000, 342000],
    public: [4356000, 1856000, 926000, 1030000, 1150000]
  }
  const sceneRois = {
    customer: [1.82, 1.88, 1.76, 1.84, 1.92],
    product: [1.39, 1.45, 1.41, 1.48, 1.54],
    channel: [2.05, 2.12, 2.05, 2.18, 2.31],
    public: [0, 0, 0, 0, 0]
  }
  const sceneRevenues = {
    customer: [2006000, 879840, 249920, 290720, 357120],
    product: [2262920, 1041100, 558360, 683760, 813120],
    channel: [4650600, 2095120, 565800, 649640, 790020],
    public: [0, 0, 0, 0, 0]
  }
  return trendPeriods.map((p, i) => ({
    period: p,
    amount: sceneAmounts[props.activeScene][i],
    roi: sceneRois[props.activeScene][i],
    revenue: sceneRevenues[props.activeScene][i]
  }))
})

const roiMax = computed(() => Math.max(...periodTrend.value.filter(p => p.roi > 0).map(p => p.roi)))
const roiMin = computed(() => Math.min(...periodTrend.value.filter(p => p.roi > 0).map(p => p.roi)))
const roiDelta = computed(() => {
  const vals = periodTrend.value.filter(p => p.roi > 0).map(p => p.roi)
  return (vals[vals.length - 1] - vals[0]).toFixed(2)
})

// ============================================================
// 优化建议：根据场景+维度动态生成
// ============================================================
const recommendations = computed(() => {
  const s = props.scenes[props.activeScene]
  const dim = activeDimension.value
  const roi = s.roi.roi || 0

  const base = []
  if (dim === 'customer' || dim === 'management') {
    // 客户视角建议
    if (roi >= 1.8) {
      base.push({ icon: '📈', title: '加大高价值客群投入', priority: 'high', detail: `A03高净值客群ROI达${roi}x，建议追加¥30-50万客户经营投入，重点在私行权益与专属服务，预估ROI提升至2.1x以上。`, actions: ['增加沙龙频次至每月2场', '提升私行客户权益预算', '配置专属客户经理'] })
    }
    if (roi < 1.5) {
      base.push({ icon: '🔄', title: '客群分层精细化运营', priority: 'warn', detail: `当前客群ROI仅${roi}x，低于行内均值1.65x，建议对A01基础客群进行分层筛选，提升高价值客户触达效率。`, actions: ['建立客户价值分层模型', '差异化礼品与权益体系', '低效客群资源退出机制'] })
    }
    if (props.activeScene === 'customer') {
      base.push({ icon: '⚙️', title: '优化管户工时归集效率', priority: 'info', detail: `本期客户经理张岚管户工时[48h]占比32%，管户AUM占比62%，工时分摊偏差+30%。建议CRM系统改造后自动同步工时数据，消除人工录入延迟。`, actions: ['CRM系统工时自动登记', 'AUM占比实时计算', '归集结果月度核对'] })
    }
  }
  if (dim === 'product' || dim === 'management') {
    // 产品视角建议
    if (roi >= 1.5) {
      base.push({ icon: '🚀', title: '产品策略维持+复制推广', priority: 'high', detail: `${s.product?.split('/')[0]?.trim() || '该产品'}ROI达${roi}x，表现优于行内均值，建议将成功经验复制至其他产品线。`, actions: ['经验复制至稳利盈系列', '建立产品ROI追踪机制', '定期产品组合优化评审'] })
    }
    if (roi < 1.5) {
      base.push({ icon: '📊', title: '产品成本结构诊断', priority: 'warn', detail: `产品ROI仅${roi}x，成本收入比${s.costIncomeRatio || '偏高'}。建议拆解成本结构，识别可压缩的推广费与激励费。`, actions: ['按渠道拆解成本明细', '建立成本上限预警', '对比同类产品成本结构'] })
    }
  }
  if (dim === 'channel' || dim === 'management') {
    // 渠道视角建议
    base.push({ icon: '📱', title: '渠道效能差异化资源配置', priority: dim === 'channel' ? 'high' : 'info', detail: '手机银行ROI达3.2x，营业网点ROI仅1.8x，差距1.78倍。建议将网点部分成本转移至手机银行数字化运营。', actions: ['网点成本压缩10%转移至线上', '增加手机银行营销预算', '网点智能化改造降成本'] })
    base.push({ icon: '🔧', title: '核心系统改造优先推进', priority: 'info', detail: '渠道交易量从核心业务系统自动拉取是驱动因子准确分摊的前提。建议试点期优先完成核心系统（改造）接口开发。', actions: ['核心系统交易量API开发', 'IT监控系统设备台账同步', '资产管理系统网点数据接入'] })
  }
  if (dim === 'management') {
    // 管理视角特有建议
    if (s.amount > 1000000) {
      base.push({ icon: '🏛️', title: '公共成本池瘦身', priority: 'warn', detail: `公共管理支撑成本达${props.money(s.amount)}，其中${s.methodDetail?.pool?.ratio || 45}%通过公共池分配。建议重新审视可直归/服务归集部分，压缩公共池比例至30%以下。`, actions: ['逐笔审查公共池来源', '扩大直接归集范围', '建立公共成本削减KPI'] })
    }
    base.push({ icon: '📋', title: '条线EVA评价机制建立', priority: 'info', detail: `当前公共成本分摊至各条线后，普惠金融部EVA接近盈亏平衡。建议建立基于EVA的条线评价体系，将成本归集结果直接纳入条线绩效考核。`, actions: ['EVA口径确认与系统改造', '条线EVA月度通报机制', '公共成本分摊合理性评审'] })
  }
  return base.slice(0, 3)
})

// ============================================================
// 模拟滑块：调节成本投入看ROI变化
// ============================================================
const simulateAmount = ref(0)
const simulateEnabled = ref(false)

const simulateRoi = computed(() => {
  if (!simulateEnabled.value) return null
  const s = props.scenes[props.activeScene]
  const revenue = s.roi.revenue || 0
  if (revenue === 0) return null
  return (revenue / (simulateAmount.value || s.amount)).toFixed(2)
})

const currentScene = computed(() => props.scenes[props.activeScene])

// ============================================================
// 四分法结构数据
// ============================================================
const methodBreakdown = computed(() => {
  const md = currentScene.value.methodDetail
  if (!md) return []
  const total = currentScene.value.amount || 1
  let cumulative = 0
  return Object.entries(md).map(([key, val], idx) => {
    const start = cumulative
    cumulative += (val.amount / total) * 100
    const labels = { direct: '第一分法\n直接归集', service: '第二分法\n服务关系', driver: '第三分法\n驱动因子', pool: '第四分法\n公共池' }
    const colors = { direct: '#49dcb1', service: '#55c8ff', driver: '#f7c46a', pool: '#e879f9' }
    return {
      key, ...val,
      label: labels[key],
      color: colors[key],
      pct: val.ratio,
      barStart: start,
      barEnd: cumulative,
      barWidth: (val.amount / total) * 100
    }
  })
})

// ROI颜色判定
const roiClass = (r) => r >= 2 ? 'roi-excellent' : r >= 1.5 ? 'roi-good' : r >= 1 ? 'roi-fair' : 'roi-poor'
const roiLabel = (r) => r >= 2 ? '优秀' : r >= 1.5 ? '良好' : r >= 1 ? '一般' : '亏损'
</script>

<template>
  <section class="page-grid roi-layout">

    <!-- ==================== 场景+维度双重选择 ==================== -->
    <article class="card panel scene-dim-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">经营ROI分析</p>
          <h3>{{ currentScene.label }} — 投入产出分析</h3>
        </div>
        <div class="period-badge">
          <span>&#9678;</span>
          <span>{{ activePeriod }}</span>
        </div>
      </div>

      <!-- 场景标签 -->
      <div class="scene-tabs">
        <button
          v-for="(scene, key) in scenes" :key="key"
          class="scene-tab"
          :class="{ active: activeScene === key }"
          @click="emit('change-scene', key)"
        >
          <span class="scene-tab-label">{{ scene.label.replace('经营', '').replace('销售', '').replace('支撑', '').replace('管理', '') }}</span>
          <strong>{{ money(scene.amount) }}</strong>
          <small>{{ scene.roi.roi > 0 ? scene.roi.roi + 'x' : '—' }}</small>
        </button>
      </div>

      <!-- 维度切换 -->
      <div class="dim-tabs">
        <button
          v-for="dim in dimensions" :key="dim.id"
          class="dim-tab"
          :class="{ active: activeDimension === dim.id }"
          @click="activeDimension = dim.id"
        >
          <span class="dim-icon">{{ dim.icon }}</span>
          <strong>{{ dim.label }}</strong>
        </button>
      </div>
      <p class="dim-focus-hint">
        <span>&#9656;</span>
        {{ dimensions.find(d => d.id === activeDimension)?.focus }}
      </p>
    </article>

    <!-- ==================== KPI卡片组 ==================== -->
    <article class="card panel kpi-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">关键指标</p>
          <h3>当前{{ dimensions.find(d => d.id === activeDimension)?.label }}KPI</h3>
        </div>
        <span class="mini">归集结果汇总</span>
      </div>
      <div class="kpi-grid">
        <div
          v-for="kpi in sceneAnalysisData[activeDimension]?.kpis || []"
          :key="kpi.label"
          class="kpi-card"
        >
          <span class="kpi-label">{{ kpi.label }}</span>
          <strong class="kpi-value">{{ kpi.value }}</strong>
          <small class="kpi-sub">{{ kpi.sub }}</small>
        </div>
      </div>
    </article>

    <!-- ==================== 趋势分析 ==================== -->
    <article class="card panel trend-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">历史趋势</p>
          <h3>ROI与成本趋势回看</h3>
        </div>
        <div class="trend-summary">
          <span :class="Number(roiDelta) >= 0 ? 'trend-up' : 'trend-down'">
            {{ Number(roiDelta) >= 0 ? '↑' : '↓' }} {{ Math.abs(roiDelta) }}x
          </span>
          <small>同比ROI变化</small>
        </div>
      </div>
      <div class="trend-chart">
        <div class="trend-bars">
          <div
            v-for="pt in periodTrend" :key="pt.period"
            class="trend-bar-group"
          >
            <div class="trend-bar-wrap">
              <div
                class="trend-bar roi-bar"
                :class="roiClass(pt.roi)"
                :style="{ height: pt.roi > 0 ? (pt.roi / roiMax * 100) + '%' : '0%' }"
                :title="pt.roi > 0 ? 'ROI: ' + pt.roi + 'x' : '—'"
              >
                <span class="bar-val" v-if="pt.roi > 0">{{ pt.roi }}x</span>
              </div>
            </div>
            <span class="trend-label">{{ pt.period }}</span>
            <span class="trend-amt">{{ money(pt.amount) }}</span>
          </div>
        </div>
      </div>
      <!-- ROI质量指示器 -->
      <div class="roi-scale">
        <span class="scale-item roi-excellent">优秀 ≥2.0x</span>
        <span class="scale-item roi-good">良好 ≥1.5x</span>
        <span class="scale-item roi-fair">一般 ≥1.0x</span>
        <span class="scale-item roi-poor">亏损 &lt;1.0x</span>
      </div>
    </article>

    <!-- ==================== 四分法成本结构 ==================== -->
    <article class="card panel structure-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">成本结构</p>
          <h3>四分法成本构成分析</h3>
        </div>
        <span class="mini">{{ money(currentScene.amount) }} 总成本</span>
      </div>
      <div class="method-breakdown">
        <div
          v-for="m in methodBreakdown"
          :key="m.key"
          class="method-item"
        >
          <div class="method-info">
            <div class="method-name-row">
              <span class="method-dot" :style="{ background: m.color }"></span>
              <strong>{{ m.label.replace('\n', '·') }}</strong>
              <span class="method-rule">{{ currentScene.methodDetail?.[m.key]?.rule?.replace('公共成本池', '公共池').replace('公共池', '') || '—' }}</span>
            </div>
            <div class="method-bar-bg">
              <div
                class="method-bar-fill"
                :style="{ width: m.pct + '%', background: m.color }"
              ></div>
            </div>
          </div>
          <div class="method-stats">
            <span class="method-pct">{{ m.pct }}%</span>
            <span class="method-amt">{{ money(m.amount) }}</span>
          </div>
        </div>
      </div>
      <div class="method-note">
        <span>&#9656;</span>
        <small>{{ activeDimension === 'customer' ? '直接归集比例越高，成本归集精准度越好。' : '' }}{{ activeDimension === 'channel' ? '驱动因子分摊比例高，说明成本与业务量高度挂钩，分配逻辑清晰。' : '' }}{{ activeDimension === 'management' ? '公共池比例过高（>40%）会影响条线成本真实性，建议逐步压缩。' : '' }}{{ activeDimension === 'product' ? '产品直归比例低时，需检查产品编码字段是否在费用报销中填写完整。' : '' }}</small>
      </div>
    </article>

    <!-- ==================== 维度对比明细 ==================== -->
    <article class="card panel compare-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">横向对比</p>
          <h3>
            {{
              activeDimension === 'customer' ? '客群投入产出对比' :
              activeDimension === 'product' ? '产品投入产出对比' :
              activeDimension === 'channel' ? '渠道效能对比' :
              '条线盈利对比'
            }}
          </h3>
        </div>
        <span class="mini">{{ dimensions.find(d => d.id === activeDimension)?.compareLabel }}</span>
      </div>

      <!-- 客户对比 -->
      <template v-if="activeDimension === 'customer'">
        <table class="compare-table">
          <thead>
            <tr>
              <th>客群名称</th>
              <th>客户数</th>
              <th>总AUM</th>
              <th>收入贡献</th>
              <th>成本投入</th>
              <th>ROI</th>
              <th>趋势</th>
              <th>评价</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="seg in sceneAnalysisData.customer.segments" :key="seg.name" :class="{ 'row-highlight': seg.roi >= 1.8 }">
              <td><strong>{{ seg.name }}</strong></td>
              <td>{{ typeof seg.clients === 'number' ? seg.clients.toLocaleString() : seg.clients }}</td>
              <td>{{ seg.aum }}</td>
              <td>{{ money(seg.revenue) }}</td>
              <td>{{ money(seg.cost) }}</td>
              <td><span class="roi-badge" :class="roiClass(seg.roi)">{{ seg.roi }}x</span></td>
              <td>
                <span class="trend-arrow" :class="seg.trend">
                  {{ seg.trend === 'up' ? '↑' : seg.trend === 'down' ? '↓' : '→' }}
                </span>
              </td>
              <td><span class="status-badge" :class="seg.status === '优' ? 'status-excellent' : seg.status === '良' ? 'status-good' : 'status-warn'">{{ seg.status }}</span></td>
            </tr>
          </tbody>
        </table>
      </template>

      <!-- 产品对比 -->
      <template v-else-if="activeDimension === 'product'">
        <table class="compare-table">
          <thead>
            <tr>
              <th>产品名称</th>
              <th>收入贡献</th>
              <th>归集成本</th>
              <th>成本收入比</th>
              <th>ROI</th>
              <th>排名</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="p in sceneAnalysisData.product.products" :key="p.name" :class="{ 'row-highlight': p.rank === 1 }">
              <td><strong>{{ p.name }}</strong></td>
              <td>{{ money(p.revenue) }}</td>
              <td>{{ money(p.cost) }}</td>
              <td>{{ p.revenue > 0 ? ((p.cost / p.revenue) * 100).toFixed(1) + '%' : '—' }}</td>
              <td><span class="roi-badge" :class="roiClass(p.roi)">{{ p.roi }}x</span></td>
              <td><span class="rank-badge" :class="'rank-' + p.rank">第{{ p.rank }}名</span></td>
            </tr>
          </tbody>
        </table>
      </template>

      <!-- 渠道对比 -->
      <template v-else-if="activeDimension === 'channel'">
        <div class="channel-bars">
          <div v-for="ch in sceneAnalysisData.channel.channels" :key="ch.name" class="channel-item">
            <div class="channel-meta">
              <span class="channel-name">{{ ch.name }}</span>
              <span class="channel-stats">
                {{ ch.txn.toLocaleString() }}笔 · {{ ch.amount }}
                <span class="roi-badge ml-8" :class="roiClass(ch.roi)">{{ ch.roi }}x</span>
              </span>
            </div>
            <div class="channel-bar-bg">
              <div
                class="channel-bar-fill"
                :style="{ width: ch.fill + '%', background: ch.roi >= 2 ? '#49dcb1' : ch.roi >= 1.5 ? '#f7c46a' : '#ff6b6b' }"
              ></div>
            </div>
            <div class="channel-status">
              <span class="status-badge" :class="ch.status === '最优' ? 'status-excellent' : ch.status === '良好' ? 'status-good' : ch.status === '一般' ? 'status-warn' : 'status-poor'">{{ ch.status }}</span>
              <span class="trend-arrow" :class="ch.trend">{{ ch.trend === 'up' ? '↑' : ch.trend === 'down' ? '↓' : '→' }}</span>
            </div>
          </div>
        </div>
      </template>

      <!-- 管理视角对比 -->
      <template v-else>
        <table class="compare-table">
          <thead>
            <tr>
              <th>条线名称</th>
              <th>条线收入</th>
              <th>条线成本</th>
              <th>条线EVA</th>
              <th>ROI</th>
              <th>成本占比</th>
              <th>状态</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="ln in sceneAnalysisData.management.lines" :key="ln.name" :class="{ 'row-highlight': ln.eva > 0 }">
              <td><strong>{{ ln.name }}</strong></td>
              <td>{{ money(ln.revenue) }}</td>
              <td>{{ money(ln.cost) }}</td>
              <td :class="ln.eva >= 0 ? 'text-teal' : 'text-red'">{{ money(Math.abs(ln.eva)) }}{{ ln.eva < 0 ? '(亏损)' : '' }}</td>
              <td><span class="roi-badge" :class="roiClass(ln.roi)">{{ ln.roi }}x</span></td>
              <td>{{ ln.ratio }}%</td>
              <td><span class="status-badge" :class="ln.status === '盈利' ? 'status-excellent' : 'status-warn'">{{ ln.status }}</span></td>
            </tr>
          </tbody>
        </table>
      </template>
    </article>

    <!-- ==================== 经营优化建议 ==================== -->
    <article class="card panel recommendation-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">决策支持</p>
          <h3>基于归集结果的优化建议</h3>
        </div>
        <div class="sim-toggle">
          <button
            class="sim-btn"
            :class="{ active: simulateEnabled }"
            @click="simulateEnabled = !simulateEnabled"
          >
            {{ simulateEnabled ? '退出模拟' : '成本模拟' }}
          </button>
        </div>
      </div>

      <!-- 模拟面板 -->
      <div v-if="simulateEnabled" class="sim-panel">
        <div class="sim-header">
          <span>成本投入模拟</span>
          <small>拖动滑块调整成本，观察ROI变化</small>
        </div>
        <div class="sim-slider">
          <input
            type="range"
            :min="Math.floor((currentScene.amount || 0) * 0.5)"
            :max="Math.floor((currentScene.amount || 0) * 2)"
            v-model.number="simulateAmount"
          />
          <div class="sim-values">
            <span>{{ money(simulateAmount) }}</span>
            <span class="sim-roi" :class="roiClass(Number(simulateRoi))" v-if="simulateRoi">
              ROI: {{ simulateRoi }}x
            </span>
          </div>
        </div>
        <div class="sim-note" v-if="simulateRoi">
          <small>
            收入固定为{{ money(currentScene.roi?.revenue || 0) }}，
            成本{{ simulateAmount < currentScene.amount ? '压缩' : '增加' }}至{{ money(simulateAmount) }}，
            ROI{{ Number(simulateRoi) > currentScene.roi.roi ? '提升' : '下降' }}至{{ simulateRoi }}x
          </small>
        </div>
      </div>

      <!-- 建议列表 -->
      <div class="rec-list">
        <div
          v-for="(rec, idx) in recommendations"
          :key="idx"
          class="rec-item"
          :class="'rec-' + rec.priority"
        >
          <div class="rec-head">
            <span class="rec-icon">{{ rec.icon }}</span>
            <strong class="rec-title">{{ rec.title }}</strong>
            <span class="rec-priority-tag" :class="'tag-' + rec.priority">
              {{ rec.priority === 'high' ? '优先执行' : rec.priority === 'warn' ? '重点关注' : '建议参考' }}
            </span>
          </div>
          <p class="rec-detail">{{ rec.detail }}</p>
          <div class="rec-actions">
            <span
              v-for="(action, ai) in rec.actions"
              :key="ai"
              class="action-chip"
            >{{ action }}</span>
          </div>
        </div>
        <div v-if="recommendations.length === 0" class="rec-empty">
          <span>—</span>
          <small>当前场景在此维度下暂无建议</small>
        </div>
      </div>
    </article>

  </section>
</template>

<style scoped>
.roi-layout {
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 16px;
  align-items: start;
}

/* === 共用标题样式 === */
.title-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 16px;
}
.title-row h3 { font-size: 15px; font-weight: 600; margin: 0; }
.eyebrow {
  font-size: 11px; color: var(--muted); text-transform: uppercase;
  letter-spacing: .5px; margin: 0 0 2px;
}
.mini { font-size: 11px; color: var(--muted); align-self: center; }

/* === 场景+维度面板 === */
.scene-dim-panel { grid-column: 1 / -1; }
.scene-tabs {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  margin-bottom: 14px;
}
.scene-tab {
  display: flex; flex-direction: column; align-items: center;
  gap: 2px; padding: 12px 8px;
  border-radius: 12px; border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text); cursor: pointer;
  transition: all .18s ease;
}
.scene-tab:hover { border-color: var(--cyan); }
.scene-tab.active { border-color: var(--cyan); background: rgba(85,200,255,.08); }
.scene-tab-label { font-size: 12px; color: var(--muted); }
.scene-tab strong { font-size: 16px; }
.scene-tab small { font-size: 11px; color: var(--teal); }
.dim-tabs {
  display: flex; gap: 8px; margin-bottom: 10px;
}
.dim-tab {
  display: flex; align-items: center; gap: 6px;
  padding: 8px 14px; border-radius: 10px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text); cursor: pointer;
  font-size: 13px; transition: all .15s ease;
}
.dim-tab:hover { border-color: var(--line-strong); }
.dim-tab.active { border-color: var(--cyan); background: rgba(85,200,255,.1); }
.dim-icon { font-size: 16px; }
.dim-focus-hint {
  font-size: 12px; color: var(--cyan);
  margin: 0; display: flex; align-items: center; gap: 6px;
}
.dim-focus-hint span { font-size: 10px; }

/* === KPI面板 === */
.kpi-panel { grid-column: 1 / -1; }
.kpi-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
}
.kpi-card {
  padding: 12px 14px; border-radius: 12px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  text-align: center;
}
.kpi-label { display: block; font-size: 11px; color: var(--muted); margin-bottom: 6px; }
.kpi-value { display: block; font-size: 16px; font-weight: 700; color: var(--text); margin-bottom: 4px; }
.kpi-sub { display: block; font-size: 10px; color: var(--muted); }

/* === 趋势面板 === */
.trend-panel { grid-column: 1 / -1; }
.trend-summary {
  display: flex; flex-direction: column; align-items: flex-end;
}
.trend-summary span { font-size: 18px; font-weight: bold; }
.trend-summary small { font-size: 11px; color: var(--muted); }
.trend-up { color: var(--teal); }
.trend-down { color: #ff6b6b; }

.trend-chart { margin: 8px 0 12px; }
.trend-bars {
  display: flex; gap: 8px; align-items: flex-end;
  height: 120px;
}
.trend-bar-group {
  flex: 1; display: flex; flex-direction: column;
  align-items: center; height: 100%;
}
.trend-bar-wrap {
  flex: 1; width: 100%;
  display: flex; align-items: flex-end;
  justify-content: center;
}
.trend-bar {
  width: 80%; border-radius: 6px 6px 0 0;
  transition: height .4s ease; position: relative;
  min-height: 4px;
}
.bar-val {
  position: absolute; top: -20px; left: 50%;
  transform: translateX(-50%);
  font-size: 10px; font-weight: bold; white-space: nowrap;
}
.trend-label { font-size: 10px; color: var(--muted); margin-top: 6px; }
.trend-amt { font-size: 9px; color: var(--muted); }

.roi-excellent { background: #49dcb1; }
.roi-good { background: #f7c46a; }
.roi-fair { background: #55c8ff; }
.roi-poor { background: #ff6b6b; }

.roi-scale {
  display: flex; gap: 16px; flex-wrap: wrap;
  padding-top: 8px; border-top: 1px solid var(--line);
}
.scale-item { font-size: 11px; padding: 3px 8px; border-radius: 999px; }
.roi-excellent { background: rgba(73,220,177,.15); color: var(--teal); }
.roi-good { background: rgba(247,196,106,.15); color: var(--gold); }
.roi-fair { background: rgba(85,200,255,.12); color: var(--cyan); }
.roi-poor { background: rgba(255,107,107,.12); color: #ff6b6b; }

/* === 成本结构 === */
.structure-panel {}
.method-breakdown { display: grid; gap: 10px; margin-bottom: 10px; }
.method-item {
  display: flex; align-items: center; gap: 12px;
  padding: 10px 12px; border-radius: 10px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.method-info { flex: 1; min-width: 0; }
.method-name-row {
  display: flex; align-items: center; gap: 8px;
  margin-bottom: 6px;
}
.method-dot { width: 8px; height: 8px; border-radius: 50%; flex-shrink: 0; }
.method-name-row strong { font-size: 12px; white-space: nowrap; }
.method-rule { font-size: 10px; color: var(--muted); overflow: hidden; text-overflow: ellipsis; }
.method-bar-bg {
  height: 6px; border-radius: 999px;
  background: rgba(255,255,255,.05);
}
.method-bar-fill {
  height: 100%; border-radius: 999px;
  transition: width .5s ease;
}
.method-stats { display: flex; flex-direction: column; align-items: flex-end; gap: 2px; flex-shrink: 0; }
.method-pct { font-size: 14px; font-weight: bold; }
.method-amt { font-size: 11px; color: var(--muted); }
.method-note {
  display: flex; align-items: flex-start; gap: 6px;
  font-size: 11px; color: var(--cyan); padding: 8px;
  background: rgba(85,200,255,.05);
  border-radius: 8px;
}
.method-note span { font-size: 9px; flex-shrink: 0; margin-top: 1px; }
.method-note small { line-height: 1.6; }

/* === 对比明细表 === */
.compare-panel { }
.compare-table {
  width: 100%; border-collapse: collapse; font-size: 12px;
}
.compare-table th {
  text-align: left; padding: 8px 10px;
  color: var(--muted); font-size: 11px;
  border-bottom: 1px solid var(--line); font-weight: normal;
}
.compare-table td {
  padding: 9px 10px;
  border-bottom: 1px solid rgba(255,255,255,.03);
}
.compare-table tr:last-child td { border-bottom: none; }
.compare-table tr:hover td { background: rgba(255,255,255,.02); }
.row-highlight td { background: rgba(73,220,177,.04) !important; }

.roi-badge {
  display: inline-block; padding: 3px 8px;
  border-radius: 999px; font-size: 11px; font-weight: bold;
}
.roi-badge.roi-excellent { background: rgba(73,220,177,.2); color: var(--teal); }
.roi-badge.roi-good { background: rgba(247,196,106,.2); color: var(--gold); }
.roi-badge.roi-fair { background: rgba(85,200,255,.15); color: var(--cyan); }
.roi-badge.roi-poor { background: rgba(255,107,107,.15); color: #ff6b6b; }

.status-badge {
  display: inline-block; padding: 3px 8px;
  border-radius: 999px; font-size: 11px;
}
.status-excellent { background: rgba(73,220,177,.15); color: var(--teal); }
.status-good { background: rgba(85,200,255,.12); color: var(--cyan); }
.status-warn { background: rgba(247,196,106,.15); color: var(--gold); }
.status-poor { background: rgba(255,107,107,.12); color: #ff6b6b; }

.rank-badge {
  display: inline-block; padding: 3px 8px;
  border-radius: 999px; font-size: 11px;
}
.rank-1 { background: rgba(73,220,177,.2); color: var(--teal); }
.rank-2 { background: rgba(247,196,106,.2); color: var(--gold); }
.rank-3 { background: rgba(85,200,255,.12); color: var(--cyan); }

.trend-arrow { font-size: 14px; }
.trend-arrow.up { color: var(--teal); }
.trend-arrow.down { color: #ff6b6b; }
.trend-arrow.stable { color: var(--muted); }

/* 渠道条形图 */
.channel-bars { display: grid; gap: 10px; }
.channel-item {
  padding: 10px 12px; border-radius: 10px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.channel-meta {
  display: flex; justify-content: space-between;
  align-items: center; margin-bottom: 6px;
}
.channel-name { font-size: 13px; font-weight: 600; }
.channel-stats { font-size: 11px; color: var(--muted); }
.ml-8 { margin-left: 8px; }
.channel-bar-bg {
  height: 8px; border-radius: 999px;
  background: rgba(255,255,255,.05); margin-bottom: 6px;
}
.channel-bar-fill {
  height: 100%; border-radius: 999px;
  transition: width .5s ease;
}
.channel-status {
  display: flex; justify-content: space-between; align-items: center;
}

.text-teal { color: var(--teal); }
.text-red { color: #ff6b6b; }

/* === 优化建议 === */
.recommendation-panel { grid-column: 1 / -1; }
.rec-list { display: grid; gap: 10px; }
.rec-item {
  padding: 14px; border-radius: 12px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  border-left-width: 3px;
}
.rec-high { border-left-color: var(--teal); }
.rec-warn { border-left-color: var(--gold); }
.rec-info { border-left-color: var(--cyan); }
.rec-head {
  display: flex; align-items: center; gap: 8px;
  margin-bottom: 8px;
}
.rec-icon { font-size: 16px; }
.rec-title { font-size: 14px; flex: 1; }
.rec-priority-tag {
  padding: 3px 10px; border-radius: 999px;
  font-size: 11px;
}
.tag-high { background: rgba(73,220,177,.15); color: var(--teal); }
.tag-warn { background: rgba(247,196,106,.15); color: var(--gold); }
.tag-info { background: rgba(85,200,255,.12); color: var(--cyan); }
.rec-detail {
  font-size: 12px; color: var(--muted); line-height: 1.7;
  margin: 0 0 10px;
}
.rec-actions { display: flex; gap: 6px; flex-wrap: wrap; }
.action-chip {
  padding: 3px 10px; border-radius: 999px;
  background: rgba(255,255,255,.05);
  border: 1px solid var(--line);
  font-size: 11px; color: var(--text);
}
.rec-empty {
  text-align: center; padding: 24px;
  color: var(--muted);
}

/* === 模拟面板 === */
.sim-toggle { }
.sim-btn {
  padding: 6px 14px; border-radius: 999px;
  border: 1px solid var(--cyan);
  background: transparent; color: var(--cyan);
  font-size: 12px; cursor: pointer;
  transition: all .15s ease;
}
.sim-btn:hover { background: rgba(85,200,255,.1); }
.sim-btn.active { background: rgba(85,200,255,.15); }
.sim-panel {
  padding: 14px; border-radius: 12px;
  background: rgba(85,200,255,.05);
  border: 1px solid rgba(85,200,255,.2);
  margin-bottom: 12px;
}
.sim-header {
  display: flex; justify-content: space-between;
  align-items: center; margin-bottom: 10px;
}
.sim-header span { font-size: 13px; font-weight: 600; }
.sim-header small { font-size: 11px; color: var(--muted); }
.sim-slider { margin-bottom: 8px; }
.sim-slider input[type="range"] {
  width: 100%; accent-color: var(--cyan);
  margin-bottom: 6px;
}
.sim-values {
  display: flex; justify-content: space-between; align-items: center;
}
.sim-values span:first-child { font-size: 16px; font-weight: bold; }
.sim-roi { font-size: 14px; font-weight: bold; }
.sim-note {
  padding: 6px 8px; border-radius: 6px;
  background: rgba(255,255,255,.03);
}
.sim-note small { font-size: 11px; color: var(--muted); }

/* period badge */
.period-badge {
  display: flex; align-items: center; gap: 6px;
  padding: 5px 10px; border-radius: 999px;
  background: rgba(73,220,177,.1);
  border: 1px solid rgba(73,220,177,.3);
  font-size: 12px; color: var(--teal); flex-shrink: 0;
}
.period-badge span:first-child {
  font-size: 8px; animation: blink 2s ease-in-out infinite;
}
@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: .3; }
}

/* 响应式 */
@media (max-width: 1100px) {
  .kpi-grid { grid-template-columns: repeat(3, 1fr); }
  .scene-tabs { grid-template-columns: repeat(2, 1fr); }
}
@media (max-width: 700px) {
  .kpi-grid { grid-template-columns: repeat(2, 1fr); }
}
</style>
