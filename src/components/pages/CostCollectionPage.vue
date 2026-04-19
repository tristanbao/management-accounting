<script setup>
import { computed } from 'vue'

const props = defineProps({
  scenes: { type: Object, required: true },
  fourCategories: { type: Array, required: true },
  activeScene: { type: String, required: true },
  activeMethod: { type: String, required: true },
  collectForm: { type: Object, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-scene', 'change-method', 'apply-scene'])

const selectedMethod = computed(() =>
  props.fourCategories?.find(c => c.id === props.activeMethod) || props.fourCategories?.[0]
)

// 费用申请页字段（支撑第一分法）
const feeFormFields = [
  { label: '业务单据号', model: 'documentNo', type: 'input', readonly: true },
  { label: '成本科目', model: 'costSubject', type: 'input' },
  { label: '费用金额（元）', model: 'amount', type: 'number' },
  { label: '归属机构', model: 'org', type: 'input' },
  { label: '归属条线', model: 'line', type: 'input' },
  { label: '归属渠道', model: 'channel', type: 'input' },
  { label: '产品编码', model: 'productCode', type: 'input' },
  { label: '客户/客群', model: 'customerSegment', type: 'input' },
  { label: '活动/项目编号', model: 'eventCode', type: 'input' },
  { label: '责任经理/岗位', model: 'owner', type: 'input' },
  { label: '是否直归', model: 'isDirect', type: 'select', options: ['是（直连经营对象）', '否（需分摊）'] },
  { label: '业务说明', model: 'notes', type: 'textarea' }
]

// 工时登记页字段（支撑第二分法）
const workHourFields = [
  { label: '工时登记单号', model: 'documentNo', type: 'input' },
  { label: '服务方人员/岗位', model: 'serverStaff', type: 'input' },
  { label: '服务对象类型', model: 'serviceTargetType', type: 'select', options: ['客户/客群', '产品', '活动', '机构', '渠道'] },
  { label: '服务对象编码', model: 'serviceTargetCode', type: 'input' },
  { label: '服务动作描述', model: 'serviceAction', type: 'textarea' },
  { label: '服务起止时间', model: 'servicePeriod', type: 'input' },
  { label: '本次工时（小时）', model: 'workHours', type: 'number' },
  { label: '工时占比（%）', model: 'workHourRatio', type: 'number' },
  { label: '受益产品范围', model: 'benefitProducts', type: 'input' },
  { label: '受益机构范围', model: 'benefitOrgs', type: 'input' },
  { label: '是否进入成本池', model: 'enterPool', type: 'select', options: ['直接归集', '进入服务成本池'] }
]

// 驱动因子台账字段（支撑第三分法）
const driverFields = [
  { label: '成本科目', model: 'costSubject', type: 'input' },
  { label: '受益对象类型', model: 'benefitType', type: 'select', options: ['机构', '产品', '渠道', '客群', '条线'] },
  { label: '受益对象编码', model: 'benefitCode', type: 'input' },
  { label: '驱动因子类型', model: 'driverType', type: 'select', options: ['交易笔数', '交易金额', '客户数', '设备台数', '开机时长', '面积占比', '工时占比'] },
  { label: '驱动因子值（当期）', model: 'driverValue', type: 'number' },
  { label: '分摊比例（%）', model: 'ratio', type: 'number' },
  { label: '数据来源系统', model: 'dataSource', type: 'input' },
  { label: '采集频率', model: 'collectFreq', type: 'select', options: ['实时', '日', '月'] }
]

// 公共成本池字段（支撑第四分法）
const poolFields = [
  { label: '成本池单据号', model: 'documentNo', type: 'input' },
  { label: '成本池层级', model: 'poolLevel', type: 'select', options: ['总行公共池', '分行公共池', '条线公共池', '网点公共池'] },
  { label: '成本性质', model: 'poolNature', type: 'select', options: ['管理费用', '人力成本', '运营支撑', '合规风险', '品牌支撑', '科技支撑'] },
  { label: '受益范围', model: 'benefitScope', type: 'input' },
  { label: '分摊维度', model: 'poolDimension', type: 'select', options: ['按收入贡献', '按资产规模', '按网点数量', '按人数', '按面积', '按业务量'] },
  { label: '分摊规则编码', model: 'poolRuleCode', type: 'input' },
  { label: '试算金额（元）', model: 'trialAmount', type: 'number' },
  { label: '审批状态', model: 'approvalStatus', type: 'select', options: ['草稿', '试算中', '待审批', '已批准', '已执行'] }
]

const sceneTabs = [
  { id: 'customer', label: '客户经营活动', icon: '👥', desc: '高净值客户经营、客群沙龙、权益发放' },
  { id: 'product', label: '产品销售推动', icon: '💰', desc: '产品推广、佣金激励、渠道线索转化' },
  { id: 'channel', label: '渠道运营支撑', icon: '📱', desc: '网点运营、线上渠道、自助设备' },
  { id: 'public', label: '公共管理支撑', icon: '🏦', desc: '总行/分行公共费用、条线支撑' }
]

function getMethodPage(methodId) {
  const pages = {
    direct: { name: '费用申请与审批页', system: '费用报销系统（改造）', color: '#49dcb1' },
    service: { name: '客户经理工时登记页', system: 'CRM系统（新增）', color: '#55c8ff' },
    driver: { name: '驱动因子台账配置页', system: '管理会计平台（新增）', color: '#f7c46a' },
    pool: { name: '公共成本池管理页', system: '管理会计平台（新增）', color: '#e879f9' }
  }
  return pages[methodId] || pages.direct
}
</script>

<template>
  <section class="page-grid collect-layout">

    <!-- ==================== 场景选择 ==================== -->
    <article class="card panel scene-tabs-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">成本归集场景</p>
          <h3>选择业务场景</h3>
        </div>
        <span class="mini">四类场景对应四分法</span>
      </div>
      <div class="scene-tabs">
        <button
          v-for="tab in sceneTabs"
          :key="tab.id"
          class="scene-tab"
          :class="{ active: activeScene === tab.id }"
          @click="emit('change-scene', tab.id)"
        >
          <span class="scene-tab-icon">{{ tab.icon }}</span>
          <div>
            <strong>{{ tab.label }}</strong>
            <small>{{ tab.desc }}</small>
          </div>
        </button>
      </div>
      <div class="scene-summary">
        <div class="scene-summary-item">
          <span>当前场景</span>
          <strong>{{ scenes[activeScene].label }}</strong>
        </div>
        <div class="scene-summary-item">
          <span>适用归集方法</span>
          <strong>{{ scenes[activeScene].method }}</strong>
        </div>
        <div class="scene-summary-item">
          <span>本期投入金额</span>
          <strong class="highlight">{{ money(scenes[activeScene].amount) }}</strong>
        </div>
      </div>
    </article>

    <!-- ==================== 业务流程说明 ==================== -->
    <article class="card panel business-flow-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">业务系统改造</p>
          <h3>{{ scenes[activeScene].fourLabel }}归集流程</h3>
        </div>
        <span class="mini">将归集逻辑嵌入业务流程</span>
      </div>
      <div class="flow-steps">
        <div
          v-for="(flow, idx) in scenes[activeScene].businessFlow"
          :key="idx"
          class="flow-step"
        >
          <div class="step-header">
            <span class="step-no">{{ String(idx + 1).padStart(2, '0') }}</span>
            <strong>{{ flow.step }}</strong>
            <span class="chip">{{ flow.system }}</span>
          </div>
          <p class="step-action">{{ flow.action }}</p>
        </div>
      </div>
    </article>

    <!-- ==================== 四分法选择 ==================== -->
    <article class="card panel method-tabs-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">四分法选择</p>
          <h3>选择成本归集方法</h3>
        </div>
      </div>
      <div class="method-tabs">
        <button
          v-for="cat in fourCategories"
          :key="cat.id"
          class="method-tab-btn"
          :class="{ active: activeMethod === cat.id }"
          :style="{ '--cat-color': cat.color }"
          @click="emit('change-method', cat.id)"
        >
          <strong>{{ cat.label }}</strong>
          <small>{{ cat.name }}</small>
        </button>
      </div>
    </article>

    <!-- ==================== 业务录入表单 ==================== -->
    <article class="card panel form-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">{{ getMethodPage(activeMethod).system }}</p>
          <h3>{{ getMethodPage(activeMethod).name }}</h3>
        </div>
        <button class="btn-apply" @click="emit('apply-scene')">加载示例数据</button>
      </div>

      <!-- 费用申请表单（第一分法） -->
      <template v-if="activeMethod === 'direct'">
        <div class="form-grid form-grid-3">
          <label v-for="field in feeFormFields" :key="field.model">
            <span>{{ field.label }}</span>
            <input
              v-if="field.type === 'input'"
              v-model="collectForm[field.model]"
              :readonly="field.readonly"
            />
            <select v-else-if="field.type === 'select'" v-model="collectForm[field.model]">
              <option v-for="opt in field.options" :key="opt" :value="opt">{{ opt }}</option>
            </select>
            <textarea v-else-if="field.type === 'textarea'" v-model="collectForm[field.model]" rows="2" />
          </label>
        </div>
        <div class="direct-redirect-note">
          <span class="direct-tag">✓ 直归判定</span>
          <p>当"是否直归"选择"是"时，系统将费用直接写入对应客户/产品/活动台账，不再进入分摊流程。</p>
        </div>
      </template>

      <!-- 工时登记表单（第二分法） -->
      <template v-else-if="activeMethod === 'service'">
        <div class="form-grid form-grid-3">
          <label v-for="field in workHourFields" :key="field.model">
            <span>{{ field.label }}</span>
            <input
              v-if="field.type === 'input'"
              v-model="collectForm[field.model]"
              type="number"
            />
            <select v-else-if="field.type === 'select'" v-model="collectForm[field.model]">
              <option v-for="opt in field.options" :key="opt" :value="opt">{{ opt }}</option>
            </select>
            <textarea v-else-if="field.type === 'textarea'" v-model="collectForm[field.model]" rows="2" />
          </label>
        </div>
        <div class="service-relation-diagram">
          <div class="relation-node server">
            <strong>服务方</strong>
            <span>{{ collectForm.serverStaff || '客户经理/产品经理' }}</span>
          </div>
          <div class="relation-arrow">→ 工时占比 {{ collectForm.workHourRatio || 0 }}% →</div>
          <div class="relation-node target">
            <strong>被服务方</strong>
            <span>{{ collectForm.serviceTargetType || '客户/产品/活动' }}：{{ collectForm.serviceTargetCode || '待填写' }}</span>
          </div>
        </div>
      </template>

      <!-- 驱动因子表单（第三分法） -->
      <template v-else-if="activeMethod === 'driver'">
        <div class="form-grid form-grid-3">
          <label v-for="field in driverFields" :key="field.model">
            <span>{{ field.label }}</span>
            <input
              v-if="field.type === 'input'"
              v-model="collectForm[field.model]"
              type="number"
            />
            <select v-else-if="field.type === 'select'" v-model="collectForm[field.model]">
              <option v-for="opt in field.options" :key="opt" :value="opt">{{ opt }}</option>
            </select>
          </label>
        </div>
        <div class="driver-factor-display">
          <div class="driver-card" v-for="dt in fourCategories.find(c => c.id === 'driver').driverTypes.slice(0, 3)" :key="dt.type">
            <strong>{{ dt.type }}</strong>
            <small>单位：{{ dt.unit }}</small>
            <p>{{ dt.example }}</p>
          </div>
        </div>
      </template>

      <!-- 公共成本池表单（第四分法） -->
      <template v-else>
        <div class="form-grid form-grid-3">
          <label v-for="field in poolFields" :key="field.model">
            <span>{{ field.label }}</span>
            <input
              v-if="field.type === 'input'"
              v-model="collectForm[field.model]"
            />
            <select v-else-if="field.type === 'select'" v-model="collectForm[field.model]">
              <option v-for="opt in field.options" :key="opt" :value="opt">{{ opt }}</option>
            </select>
          </label>
        </div>
        <div class="pool-levels">
          <div class="pool-level-card" v-for="pl in fourCategories.find(c => c.id === 'pool').poolLevels" :key="pl.level">
            <strong>{{ pl.level }}</strong>
            <p>{{ pl.desc }}</p>
            <small>例：{{ pl.example }}</small>
          </div>
        </div>
      </template>
    </article>

    <!-- ==================== 归集方法说明 ==================== -->
    <article class="card panel method-detail-panel">
      <div class="title-row">
        <h3>归集方法说明</h3>
        <span class="mini">{{ selectedMethod.category }}</span>
      </div>
      <div class="method-detail-box" :style="{ borderColor: selectedMethod.color }">
        <div class="method-detail-head">
          <strong>{{ selectedMethod.name }}</strong>
          <span class="chip" :style="{ background: selectedMethod.color + '22', color: selectedMethod.color }">{{ selectedMethod.sourceSystem }}</span>
        </div>
        <p class="method-desc">{{ selectedMethod.desc }}</p>
        <div class="method-core-logic">
          <strong>核心逻辑：</strong>
          <p>{{ selectedMethod.coreLogic }}</p>
        </div>
      </div>
      <div class="fields-required">
        <h4>新增/改造录入字段</h4>
        <div class="field-tags">
          <div v-for="f in selectedMethod.fields" :key="f.tag" class="field-tag" :class="{ required: f.required }">
            <strong>{{ f.name }}</strong>
            <span>{{ f.tag }}</span>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 典型成本示例 ==================== -->
    <article class="card panel typical-cost-panel">
      <div class="title-row">
        <h3>典型成本归集示例</h3>
        <span class="mini">{{ selectedMethod.businessPage }}</span>
      </div>

      <!-- 直接归集示例 -->
      <template v-if="activeMethod === 'direct'">
        <table>
          <thead>
            <tr>
              <th>成本项目</th>
              <th>直连对象</th>
              <th>归集金额</th>
              <th>归集规则</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="tc in selectedMethod.typicalCosts" :key="tc.item">
              <td>{{ tc.item }}</td>
              <td><span class="direct-tag">{{ tc.target }}</span></td>
              <td>{{ money(tc.cost) }}</td>
              <td>FR-D001 客户直归法</td>
            </tr>
          </tbody>
        </table>
      </template>

      <!-- 服务关系示例 -->
      <template v-else-if="activeMethod === 'service'">
        <table>
          <thead>
            <tr>
              <th>成本项目</th>
              <th>服务关系</th>
              <th>服务方</th>
              <th>归集金额</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="tc in selectedMethod.typicalCosts" :key="tc.item">
              <td>{{ tc.item }}</td>
              <td>{{ tc.target }}</td>
              <td><span class="service-tag">{{ tc.service }}</span></td>
              <td>{{ money(tc.cost) }}</td>
            </tr>
          </tbody>
        </table>
      </template>

      <!-- 驱动因子示例 -->
      <template v-else-if="activeMethod === 'driver'">
        <div v-for="tc in selectedMethod.typicalCosts" :key="tc.item" class="driver-example">
          <div class="driver-example-head">
            <strong>{{ tc.item }}</strong>
            <span class="driver-type-tag">{{ tc.driver }}</span>
          </div>
          <div class="driver-targets">
            <div v-for="t in tc.targets" :key="t.name" class="driver-target">
              <span>{{ t.name }}</span>
              <span>{{ t.ratio }}</span>
              <span>{{ t.value }}</span>
              <strong>{{ money(t.amount) }}</strong>
            </div>
          </div>
        </div>
      </template>

      <!-- 公共池示例 -->
      <template v-else>
        <table>
          <thead>
            <tr>
              <th>成本项目</th>
              <th>成本池层级</th>
              <th>分摊金额</th>
              <th>分摊规则</th>
              <th>状态</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="tc in selectedMethod.typicalCosts" :key="tc.item">
              <td>{{ tc.item }}</td>
              <td><span class="pool-tag">{{ tc.level }}</span></td>
              <td>{{ money(tc.amount) }}</td>
              <td>{{ tc.poolRule }}</td>
              <td>
                <span class="status-tag" :class="tc.status === '已分配' ? 'status-ok' : tc.status === '试算中' ? 'status-pending' : 'status-wait'">
                  {{ tc.status }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </template>
    </article>

  </section>
</template>

<style scoped>
.collect-layout {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.scene-tabs-panel { grid-column: 1 / -1; }

.scene-tabs {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 12px;
  margin-bottom: 16px;
}

.scene-tab {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  border-radius: 16px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text);
  cursor: pointer;
  text-align: left;
  transition: all .18s ease;
}

.scene-tab:hover { border-color: var(--line-strong); background: rgba(85,200,255,.06); }
.scene-tab.active {
  border-color: var(--cyan);
  background: rgba(85,200,255,.12);
}

.scene-tab-icon { font-size: 22px; }

.scene-tab strong { display: block; font-size: 14px; }
.scene-tab small { display: block; color: var(--muted); font-size: 11px; margin-top: 4px; }

.scene-summary {
  display: flex;
  gap: 24px;
  padding: 14px 16px;
  border-radius: 16px;
  background: rgba(73,220,177,.06);
  border: 1px solid var(--line);
}

.scene-summary-item { flex: 1; }
.scene-summary-item span { display: block; color: var(--muted); font-size: 12px; margin-bottom: 4px; }
.scene-summary-item strong { font-size: 13px; }
.scene-summary-item .highlight { color: var(--teal); font-size: 16px; }

/* 业务流 */
.flow-steps { display: grid; gap: 10px; }
.flow-step {
  padding: 14px 16px;
  border-radius: 14px;
  background: rgba(85,200,255,.04);
  border: 1px solid var(--line);
}
.step-header { display: flex; align-items: center; gap: 10px; margin-bottom: 8px; }
.step-no {
  min-width: 28px; height: 28px; line-height: 28px; text-align: center;
  border-radius: 50%; background: rgba(85,200,255,.15); color: var(--cyan);
  font-size: 12px; font-weight: bold;
}
.step-action { color: var(--muted); font-size: 13px; line-height: 1.6; margin: 0; }

/* 方法选择 */
.method-tabs {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
}

.method-tab-btn {
  padding: 14px 12px;
  border-radius: 14px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text);
  cursor: pointer;
  text-align: center;
  transition: all .18s ease;
}

.method-tab-btn:hover { border-color: var(--cat-color, var(--cyan)); }
.method-tab-btn.active {
  border-color: var(--cat-color, var(--cyan));
  background: color-mix(in srgb, var(--cat-color, var(--cyan)) 12%, transparent);
}

.method-tab-btn strong { display: block; font-size: 13px; color: var(--cat-color, var(--cyan)); }
.method-tab-btn small { display: block; color: var(--muted); font-size: 11px; margin-top: 4px; }

/* 表单 */
.form-panel { grid-column: 1 / -1; }
.form-grid-3 { grid-template-columns: repeat(3, minmax(0, 1fr)); }

.btn-apply {
  padding: 8px 18px;
  border-radius: 10px;
  border: 1px solid var(--line-strong);
  background: rgba(73,220,177,.12);
  color: var(--teal);
  cursor: pointer;
  font-size: 13px;
  transition: all .18s ease;
}
.btn-apply:hover { background: rgba(73,220,177,.2); }

.direct-redirect-note {
  margin-top: 14px;
  padding: 12px 14px;
  border-radius: 12px;
  background: rgba(73,220,177,.08);
  border: 1px solid rgba(73,220,177,.2);
}

.direct-tag {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(73,220,177,.2);
  color: var(--teal);
  font-size: 12px;
  margin-bottom: 8px;
}

/* 服务关系图 */
.service-relation-diagram {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-top: 14px;
  padding: 16px;
  border-radius: 14px;
  background: rgba(85,200,255,.06);
  border: 1px solid var(--line);
}

.relation-node {
  flex: 1;
  padding: 12px;
  border-radius: 12px;
  text-align: center;
}
.relation-node.server { background: rgba(85,200,255,.15); }
.relation-node.target { background: rgba(73,220,177,.15); }
.relation-node strong { display: block; font-size: 12px; color: var(--muted); margin-bottom: 4px; }
.relation-node span { font-size: 13px; }
.relation-arrow { color: var(--muted); font-size: 13px; white-space: nowrap; }

/* 驱动因子 */
.driver-factor-display {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
  margin-top: 14px;
}

.driver-card {
  padding: 12px;
  border-radius: 12px;
  background: rgba(247,196,106,.08);
  border: 1px solid rgba(247,196,106,.2);
}
.driver-card strong { font-size: 13px; color: var(--gold); }
.driver-card small { display: block; color: var(--muted); font-size: 11px; margin: 4px 0; }
.driver-card p { font-size: 12px; color: var(--muted); margin: 0; }

/* 成本池层级 */
.pool-levels {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 10px;
  margin-top: 14px;
}

.pool-level-card {
  padding: 12px;
  border-radius: 12px;
  background: rgba(232,121,249,.06);
  border: 1px solid rgba(232,121,249,.15);
}
.pool-level-card strong { font-size: 13px; color: #e879f9; }
.pool-level-card p { font-size: 12px; color: var(--muted); margin: 4px 0; }
.pool-level-card small { font-size: 11px; color: var(--muted); }

/* 方法说明 */
.method-detail-box {
  padding: 16px;
  border-radius: 14px;
  border: 1px solid;
  background: rgba(255,255,255,.02);
  margin-bottom: 16px;
}

.method-detail-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}
.method-detail-head strong { font-size: 15px; }
.method-desc { color: var(--muted); font-size: 13px; line-height: 1.7; margin-bottom: 10px; }
.method-core-logic { font-size: 13px; }
.method-core-logic strong { color: var(--cyan); }
.method-core-logic p { margin: 4px 0 0; color: var(--muted); }

.fields-required h4 { font-size: 14px; margin-bottom: 10px; }
.field-tags { display: flex; flex-wrap: wrap; gap: 8px; }
.field-tag {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 12px;
  border-radius: 999px;
  background: rgba(85,200,255,.08);
  border: 1px solid var(--line);
  font-size: 12px;
}
.field-tag.required { border-color: rgba(85,200,255,.3); }
.field-tag span { color: var(--muted); font-size: 11px; }

/* 标签样式 */
.direct-tag { background: rgba(73,220,177,.2); color: var(--teal); padding: 4px 10px; border-radius: 999px; font-size: 12px; }
.service-tag { background: rgba(85,200,255,.2); color: var(--cyan); padding: 4px 10px; border-radius: 999px; font-size: 12px; }
.driver-type-tag { background: rgba(247,196,106,.2); color: var(--gold); padding: 4px 10px; border-radius: 999px; font-size: 12px; }
.pool-tag { background: rgba(232,121,249,.2); color: #e879f9; padding: 4px 10px; border-radius: 999px; font-size: 12px; }

.status-tag {
  display: inline-block;
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 12px;
}
.status-ok { background: rgba(73,220,177,.2); color: var(--teal); }
.status-pending { background: rgba(247,196,106,.2); color: var(--gold); }
.status-wait { background: rgba(85,200,255,.15); color: var(--cyan); }

/* 典型成本 */
.typical-cost-panel { grid-column: 1 / -1; }

/* 驱动分摊示例 */
.driver-example { margin-bottom: 14px; padding: 14px; border-radius: 14px; background: rgba(247,196,106,.04); border: 1px solid rgba(247,196,106,.15); }
.driver-example-head { display: flex; align-items: center; gap: 10px; margin-bottom: 10px; }
.driver-example-head strong { font-size: 13px; }
.driver-targets { display: grid; gap: 8px; }
.driver-target {
  display: grid;
  grid-template-columns: 1fr 60px 100px 1fr;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 10px;
  background: rgba(255,255,255,.02);
  font-size: 12px;
  align-items: center;
}
.driver-target strong { text-align: right; color: var(--gold); }

@media (max-width: 1100px) {
  .scene-tabs { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .method-tabs { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .form-grid-3 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .driver-factor-display { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .pool-levels { grid-template-columns: 1fr; }
}
</style>
