<script setup>
import { computed, ref, watch } from 'vue'

const props = defineProps({
  scenes: { type: Object, required: true },
  fourCategories: { type: Array, required: true },
  activeScene: { type: String, required: true },
  activeMethod: { type: String, required: true },
  collectForm: { type: Object, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-scene', 'change-method'])

const selectedMethod = computed(() =>
  props.fourCategories?.find(c => c.id === props.activeMethod) || props.fourCategories?.[0]
)

// ============================================================
// 数据接入状态
// ============================================================
const isLoading = ref(false)
const loadProgress = ref(0)
const integrationLogs = ref([])

function addLog(type, system, message) {
  integrationLogs.value.unshift({ type, system, message, time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit', second: '2-digit' }) })
  if (integrationLogs.value.length > 20) integrationLogs.value.pop()
}

// 监听 apply-scene，在 App.vue emit 之前触发 loading 动画
function handleApply() {
  if (isLoading.value) return
  isLoading.value = true
  loadProgress.value = 0
  integrationLogs.value = []

  const method = props.activeMethod
  const scene = props.activeScene
  const scenes = {
    customer: '客户经营活动',
    product: '产品销售推动',
    channel: '渠道运营支撑',
    public: '公共管理支撑'
  }
  const sceneLabel = scenes[scene]

  const steps = getIntegrationSteps(method, scene)

  let delay = 0
  steps.forEach((step, i) => {
    setTimeout(() => {
      addLog(step.type, step.system, step.message)
      loadProgress.value = Math.round(((i + 1) / steps.length) * 100)
      if (i === steps.length - 1) {
        setTimeout(() => {
          isLoading.value = false
          applySceneData()
        }, 300)
      }
    }, delay)
    delay += step.duration
  })
}

function getIntegrationSteps(method, scene) {
  const sceneLabel = { customer: '客户经营活动', product: '产品销售推动', channel: '渠道运营支撑', public: '公共管理支撑' }[scene]

  if (method === 'direct') {
    return [
      { type: 'api', system: '费用报销系统', message: '调入费用申请接口 FY-2026-API-001，获取本期费用明细...', duration: 400 },
      { type: 'api', system: 'CRM系统', message: `查询客户经理[张岚]管户关系，获取客群编码[A03]归属信息...`, duration: 500 },
      { type: 'api', system: '产品管理系统', message: '查询产品[稳利盈6M]归属条线，获取产品编码与条线映射...', duration: 400 },
      { type: 'api', system: '营销管理平台', message: `拉取活动编号[ACT-PRIV-2026Q1-015]关联产品与客群数据...`, duration: 350 },
      { type: 'check', system: '管理会计平台', message: `直归规则[FR-D001]命中校验：客群[A03]归属明确，标记为直接归集`, duration: 300 },
      { type: 'done', system: '费用报销系统', message: `写入归集台账-客户维，写入金额 ¥186,000，状态：已归集`, duration: 300 },
    ]
  } else if (method === 'service') {
    return [
      { type: 'api', system: 'CRM系统', message: '调入客户经理工时查询接口 /crm/workhours?month=2026-04', duration: 500 },
      { type: 'api', system: '人力资源系统', message: '查询客户经理[张岚]岗位-条线映射，获取归属条线[零售金融部]...', duration: 450 },
      { type: 'api', system: 'CRM系统', message: '拉取管户关系表，管户AUM≥600万客户数[203户]，占比[62%]...', duration: 500 },
      { type: 'api', system: '产品管理系统', message: '查询产品经理[李峻]支持产品列表，支持产品[普惠经营贷/稳利盈6M]...', duration: 400 },
      { type: 'calc', system: '管理会计平台', message: '工时占比自动计算：客户经理月均标准工时[176h]，本月管户工时[48h]，占比[32%]', duration: 350 },
      { type: 'done', system: 'CRM系统', message: `写入归集台账-服务关系维，工时成本分配至客群[A03财富客群]...`, duration: 300 },
    ]
  } else if (method === 'driver') {
    // 渠道运营支撑：主要从业务系统自动拉取，非费用报销
    if (scene === 'channel') {
      return [
        { type: 'api', system: '核心业务系统（改造）', message: '调入各渠道交易量数据 /core/channel-trans?period=2026-04，交易笔数自动按渠道标识分类', duration: 500 },
        { type: 'api', system: 'IT监控系统（改造）', message: '拉取ATM/CDM设备开机日志，营业部[2,280台时]/零售支行[2,040台时]/社区银行[1,680台时]', duration: 500 },
        { type: 'api', system: '资产管理系统（改造）', message: '同步网点面积数据：总行大楼[3,200㎡]/分行办公楼[4,800㎡]/各支行[4,800㎡]', duration: 400 },
        { type: 'api', system: '财务系统', message: '查询成本科目[渠道运营费-自助设备/网点运营]，确认原始发生额[¥342,000]', duration: 400 },
        { type: 'calc', system: '管理会计平台', message: '驱动因子[FR-V002]执行：按各渠道交易量占比自动分摊，手机银行[42%]/网银[28%]/网点[30%]', duration: 400 },
        { type: 'done', system: '管理会计平台', message: `写入归集台账-驱动因子维，分摊结果：手机银行[¥143,640] / 网银[¥95,760] / 网点[¥102,600]`, duration: 300 },
      ]
    }
    const driverByScene = {
      customer: { sys: '核心业务系统', msg: '拉取2026年4月活跃客户数据，全量客户[328户]，A03客群[203户]' },
      product: { sys: 'CRM系统', msg: '拉取授信转化客户数据，当月转化客户[286户]，普惠经营贷[157户]' },
      public: { sys: '人力资源系统', msg: '拉取各条线工时记录，全行科技支持工时[1,200h]，零售条线[504h]' }
    }
    const d = driverByScene[scene]
    return [
      { type: 'api', system: '核心业务系统', message: '调入交易量数据接口 /core/transactions?period=2026-04', duration: 500 },
      { type: 'api', system: d.sys, message: d.msg, duration: 500 },
      { type: 'api', system: 'IT监控系统', message: '拉取设备台账，网点设备台数[128台]，设备分布已同步', duration: 400 },
      { type: 'calc', system: '管理会计平台', message: '驱动因子计算：受益对象[3个]，按交易量/工时比例自动分摊', duration: 400 },
      { type: 'api', system: '财务系统', message: '查询成本科目，获取原始发生额，驱动因子自动关联', duration: 350 },
      { type: 'done', system: '管理会计平台', message: `写入归集台账-驱动因子维，分摊结果已自动写入各受益对象`, duration: 300 },
    ]
  } else {
    return [
      { type: 'api', system: '财务系统（改造）', message: '公共费用原始台账直采接口 /fin/public-costs?period=2026-Q1，按成本池层级[总行/分行/条线/网点]自动分类', duration: 500 },
      { type: 'api', system: '核心业务系统（改造）', message: '查询各分行收入贡献比例[Nanchang:32%/Jiujiang:28%/Fuzhou:40%]，自动计算分摊权重', duration: 500 },
      { type: 'api', system: '人力资源系统（改造）', message: '同步各条线人数[零售:1,280人/对公:860人/普惠:540人]、岗位-条线映射数据', duration: 450 },
      { type: 'calc', system: '管理会计平台', message: '规则[FR-P001→FR-P002→FR-P003]执行"总行→分行→条线"逐级分摊，试算完成', duration: 400 },
      { type: 'api', system: '审批流系统', message: '提交分摊试算结果至审批流，审批节点[分行财务经理→总行运营部]', duration: 400 },
      { type: 'done', system: '管理会计平台', message: `写入归集台账-公共池维，分摊状态[待审批]，总行[¥575,000]→分行[Nanchang:¥184,000/Jiujiang:¥161,000]`, duration: 300 },
    ]
  }
}

// ============================================================
// 费用申请页字段（支撑第一分法）
// ============================================================
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

// 每个方法对应的外部系统接入来源
const integrationSources = computed(() => {
  const m = props.activeMethod
  const scene = props.activeScene
  const sceneLabel = { customer: '客户经营活动', product: '产品销售推动', channel: '渠道运营支撑', public: '公共管理支撑' }[scene]

  if (m === 'direct') {
    const systemsByScene = {
      customer: [
        { name: '费用报销系统', role: '费用申请原始数据来源', fields: ['documentNo', 'costSubject', 'amount', 'org', 'line'], status: '已对接', color: '#49dcb1' },
        { name: 'CRM系统', role: '客户经理与客群归属查询', fields: ['owner', 'customerSegment'], status: '已对接', color: '#55c8ff' },
        { name: '产品管理系统', role: '产品编码与条线映射', fields: ['productCode', 'channel'], status: '已对接', color: '#f7c46a' },
        { name: '权益平台', role: '客户权益领取记录归集', fields: ['eventCode', 'customerSegment'], status: '改造中', color: '#e879f9' }
      ],
      product: [
        { name: '营销管理平台', role: '产品推广计划与广告投放', fields: ['productCode', 'eventCode', 'channel'], status: '已对接', color: '#49dcb1' },
        { name: '数字营销平台', role: '广告转化线索数据拉取', fields: ['eventCode', 'conversionCount'], status: '改造中', color: '#55c8ff' },
        { name: '费用报销系统', role: '佣金激励发放数据', fields: ['documentNo', 'productCode', 'amount'], status: '已对接', color: '#f7c46a' },
        { name: 'CRM系统', role: '产品经理支持产品列表', fields: ['benefitProducts', 'serverStaff'], status: '新增中', color: '#e879f9' }
      ],
      channel: [
        { name: '核心业务系统', role: '各渠道交易笔数/金额自动同步（改造）', fields: ['driverValue', 'ratio', 'channelCode'], status: '已对接', color: '#49dcb1' },
        { name: 'IT监控系统', role: '自助设备开机时长/系统调用量日志同步（改造）', fields: ['driverValue', 'benefitCode', 'deviceUpTime'], status: '已对接', color: '#55c8ff' },
        { name: '资产管理系统', role: '网点面积/设备台账同步（改造）', fields: ['benefitType', 'benefitCode', 'branchArea'], status: '改造中', color: '#f7c46a' },
        { name: '财务系统', role: '成本科目原始发生额直采（改造）', fields: ['costSubject', 'dataSource', 'trialAmount'], status: '已对接', color: '#e879f9' }
      ],
      public: [
        { name: '财务系统', role: '公共费用原始台账直采，按成本池层级自动分类（改造）', fields: ['documentNo', 'costSubject', 'trialAmount', 'poolLevel'], status: '已对接', color: '#49dcb1' },
        { name: '核心业务系统', role: '各分行收入贡献比例自动计算（改造）', fields: ['benefitScope', 'poolDimension', 'revenueRatio'], status: '改造中', color: '#55c8ff' },
        { name: '人力资源系统', role: '各条线人数与岗位分布同步（改造）', fields: ['poolDimension', 'benefitScope', 'staffCount'], status: '改造中', color: '#f7c46a' },
        { name: '审批流系统', role: '分摊规则审批与执行状态（新增）', fields: ['approvalStatus', 'poolRuleCode'], status: '新增中', color: '#e879f9' }
      ]
    }
    const notesByScene = {
      customer: '高净值客户礼品费、权益费、沙龙活动费，在费用申请时录入客群编码，审批通过后直接写入客户维度归集台账。客户经理管户工时由CRM系统按AUM占比自动计算，无需人工录入。',
      product: '产品推广费、广告投放费在营销管理平台登记推广计划后，转化线索从数字营销平台API自动拉取，无需在费用报销系统逐笔录入。',
      channel: '渠道运营成本通过业务系统自动接入：交易量从核心业务系统拉取、设备台账从IT监控系统同步、网点面积从资产管理系统接入，按驱动因子自动分摊，全流程无需人工录入费用。',
      public: '总行/分行公共管理费从财务系统原始台账自动接入，无需人工录入；受益范围识别与分摊因子从核心业务、人力资源、资产管理系统自动拉取，按分层规则逐级分摊。'
    }
    return {
      title: `${sceneLabel} - 外部系统数据接入`,
      systems: systemsByScene[scene] || systemsByScene.customer,
      note: notesByScene[scene] || notesByScene.customer
    }
  } else if (m === 'service') {
    const systemsByScene = {
      customer: [
        { name: 'CRM系统', role: '客户经理工时登记与管户关系', fields: ['serverStaff', 'workHours', 'workHourRatio'], status: '已对接', color: '#55c8ff' },
        { name: '人力资源系统', role: '岗位-条线映射与人员归属', fields: ['serverStaff', 'benefitOrgs'], status: '已对接', color: '#49dcb1' },
        { name: '权益平台', role: '客户权益领取与客群关联', fields: ['serviceTargetCode', 'serviceAction'], status: '改造中', color: '#f7c46a' },
        { name: '管理会计平台', role: '工时占比计算与台账写入', fields: ['workHourRatio', 'serviceTargetCode'], status: '新增中', color: '#e879f9' }
      ],
      product: [
        { name: 'CRM系统', role: '产品经理支持工时登记', fields: ['serverStaff', 'workHours', 'workHourRatio'], status: '已对接', color: '#55c8ff' },
        { name: '产品管理系统', role: '产品经理支持产品列表', fields: ['benefitProducts', 'serverStaff'], status: '新增中', color: '#f7c46a' },
        { name: '人力资源系统', role: '岗位-产品归属映射', fields: ['serverStaff', 'benefitProducts'], status: '已对接', color: '#49dcb1' },
        { name: '管理会计平台', role: '服务关系计算与台账写入', fields: ['workHourRatio', 'serviceTargetCode'], status: '新增中', color: '#e879f9' }
      ],
      channel: [
        { name: 'CRM系统', role: '渠道运营人员工时登记（改造）', fields: ['serverStaff', 'workHours', 'workHourRatio'], status: '改造中', color: '#55c8ff' },
        { name: 'IT监控系统', role: '自助设备运维工时与设备运行数据（改造）', fields: ['serverStaff', 'driverValue', 'deviceUpTime'], status: '改造中', color: '#49dcb1' },
        { name: '资产管理系统', role: '设备台账与维护工时同步（改造）', fields: ['benefitCode', 'workHourRatio', 'branchArea'], status: '改造中', color: '#f7c46a' },
        { name: '管理会计平台', role: '工时占比计算与台账写入（新增）', fields: ['workHourRatio', 'serviceTargetCode'], status: '新增中', color: '#e879f9' }
      ],
      public: [
        { name: '人力资源系统', role: '各条线公共支撑工时登记', fields: ['serverStaff', 'workHours', 'workHourRatio'], status: '新增中', color: '#49dcb1' },
        { name: '财务系统', role: '公共管理费用与人力成本关联', fields: ['costSubject', 'trialAmount'], status: '已对接', color: '#55c8ff' },
        { name: '审批流系统', role: '公共成本分配审批', fields: ['approvalStatus', 'poolRuleCode'], status: '新增中', color: '#f7c46a' },
        { name: '管理会计平台', role: '管理支撑服务关系计算', fields: ['workHourRatio', 'serviceTargetCode'], status: '新增中', color: '#e879f9' }
      ]
    }
    const notesByScene = {
      customer: '客户经理管户工时按AUM占比归集至客群，CRM系统自动计算管户AUM占比，无需人工录入。权益平台改造后，客户权益领取记录按客群编码自动归集。',
      product: '产品经理支持各产品的工时占比由CRM系统自动计算，按受益产品分摊人力成本。数字营销平台改造后，广告转化线索自动拉取，无需人工填报。',
      channel: '渠道运营人员工时由CRM系统改造后自动登记；IT设备运维工时由IT监控系统改造后同步；管理会计平台按工时占比自动计算并写入归集台账。',
      public: '总行/分行公共支撑工时由人力资源系统改造后自动登记，按岗位和人数比例分配至各受益条线，需审批确认后方可生效。财务系统改造后公共费用原始台账自动同步，无需人工录入。'
    }
    return {
      title: `${sceneLabel} - 外部系统数据接入`,
      systems: systemsByScene[scene] || systemsByScene.customer,
      note: notesByScene[scene] || notesByScene.customer
    }
  } else if (m === 'driver') {
    const systemsByScene = {
      customer: [
        { name: '核心业务系统', role: '活跃客户数与客户分层数据', fields: ['driverValue', 'ratio'], status: '已对接', color: '#55c8ff' },
        { name: 'CRM系统', role: '客户经理管户AUM数据', fields: ['driverValue', 'benefitCode'], status: '已对接', color: '#49dcb1' },
        { name: '权益平台', role: '客户权益领取活跃度', fields: ['driverValue', 'ratio'], status: '改造中', color: '#f7c46a' },
        { name: '财务系统', role: '客户经营费用原始发生额', fields: ['costSubject', 'dataSource'], status: '已对接', color: '#e879f9' }
      ],
      product: [
        { name: 'CRM系统', role: '授信转化客户数与转化线索', fields: ['driverValue', 'ratio'], status: '已对接', color: '#55c8ff' },
        { name: '数字营销平台', role: '广告投放转化率数据', fields: ['driverValue', 'benefitCode'], status: '改造中', color: '#49dcb1' },
        { name: '核心业务系统', role: '产品销售额与客户数', fields: ['driverValue', 'ratio'], status: '已对接', color: '#f7c46a' },
        { name: '财务系统', role: '产品推广费用原始发生额', fields: ['costSubject', 'dataSource'], status: '已对接', color: '#e879f9' }
      ],
      channel: [
        { name: '核心业务系统', role: '各渠道交易笔数/金额自动同步（改造）', fields: ['driverValue', 'ratio', 'channelCode'], status: '已对接', color: '#55c8ff' },
        { name: 'IT监控系统', role: '设备开机时长/系统调用量/线上行为日志同步（改造）', fields: ['driverValue', 'benefitCode', 'deviceUpTime'], status: '已对接', color: '#49dcb1' },
        { name: '资产管理系统', role: '网点面积/设备台数/折旧成本同步（改造）', fields: ['benefitType', 'benefitCode', 'branchArea'], status: '改造中', color: '#f7c46a' },
        { name: '财务系统', role: '成本科目原始发生额直采（改造）', fields: ['costSubject', 'dataSource', 'trialAmount'], status: '已对接', color: '#e879f9' }
      ],
      public: [
        { name: '人力资源系统', role: '各条线工时记录与人力成本', fields: ['driverValue', 'ratio'], status: '新增中', color: '#49dcb1' },
        { name: '资产管理系统', role: '各机构面积与设备分布', fields: ['driverValue', 'benefitCode'], status: '改造中', color: '#55c8ff' },
        { name: '核心业务系统', role: '各分行收入贡献比例', fields: ['ratio', 'poolDimension'], status: '已对接', color: '#f7c46a' },
        { name: '财务系统', role: '公共管理费用原始发生额', fields: ['costSubject', 'trialAmount'], status: '已对接', color: '#e879f9' }
      ]
    }
    const notesByScene = {
      customer: '按活跃客户数将客户经营成本分摊至各客群，活跃度数据从核心业务系统和CRM系统自动拉取，无需人工录入。',
      product: '按授信转化客户数将产品推广成本分摊至各产品，转化线索从CRM和数字营销平台API自动拉取，无需人工填报。',
      channel: '按交易量、设备开机时长等驱动因子将渠道运营成本分摊至各受益渠道，数据从核心业务系统和IT监控系统自动同步，全流程无需人工录入。',
      public: '按工时占比、面积、人数等驱动因子将公共管理成本分配至各条线/机构，分摊因子从人力资源系统和资产管理系统自动拉取。'
    }
    return {
      title: `${sceneLabel} - 外部系统数据接入`,
      systems: systemsByScene[scene] || systemsByScene.customer,
      note: notesByScene[scene] || notesByScene.customer
    }
  } else {
    const systemsByScene = {
      customer: [
        { name: '财务系统', role: '客户经营相关公共费用汇总', fields: ['documentNo', 'costSubject', 'trialAmount'], status: '已对接', color: '#49dcb1' },
        { name: '核心业务系统', role: '各客群收入贡献比例计算', fields: ['benefitScope', 'poolDimension'], status: '已对接', color: '#55c8ff' },
        { name: 'CRM系统', role: '客群AUM与客户数分布', fields: ['benefitScope', 'ratio'], status: '改造中', color: '#f7c46a' },
        { name: '审批流系统', role: '客户经营支撑费用分配审批', fields: ['approvalStatus', 'poolRuleCode'], status: '新增中', color: '#e879f9' }
      ],
      product: [
        { name: '财务系统', role: '产品推广相关公共费用汇总', fields: ['documentNo', 'costSubject', 'trialAmount'], status: '已对接', color: '#49dcb1' },
        { name: '核心业务系统', role: '各产品线收入贡献比例', fields: ['benefitScope', 'poolDimension'], status: '已对接', color: '#55c8ff' },
        { name: '产品管理系统', role: '产品-条线映射关系', fields: ['benefitScope', 'poolRuleCode'], status: '改造中', color: '#f7c46a' },
        { name: '审批流系统', role: '产品推广支撑费用分配审批', fields: ['approvalStatus', 'poolRuleCode'], status: '新增中', color: '#e879f9' }
      ],
      channel: [
        { name: '财务系统', role: '渠道运营公共费用汇总', fields: ['documentNo', 'costSubject', 'trialAmount'], status: '已对接', color: '#49dcb1' },
        { name: '资产管理系统', role: '各渠道网点面积与设备分布', fields: ['benefitScope', 'poolDimension'], status: '改造中', color: '#55c8ff' },
        { name: 'IT监控系统', role: '系统调用量分布比例', fields: ['benefitScope', 'ratio'], status: '已对接', color: '#f7c46a' },
        { name: '审批流系统', role: '渠道公共费用分配审批', fields: ['approvalStatus', 'poolRuleCode'], status: '新增中', color: '#e879f9' }
      ],
      public: [
        { name: '财务系统', role: '总行/分行公共管理费用原始台账', fields: ['documentNo', 'costSubject', 'trialAmount', 'poolNature'], status: '已对接', color: '#49dcb1' },
        { name: '核心业务系统', role: '各分行收入贡献比例计算', fields: ['benefitScope', 'poolDimension'], status: '已对接', color: '#55c8ff' },
        { name: '人力资源系统', role: '各机构人数与岗位分布', fields: ['poolDimension', 'benefitScope'], status: '新增中', color: '#f7c46a' },
        { name: '审批流系统', role: '公共成本池分摊规则审批与执行状态', fields: ['approvalStatus', 'poolRuleCode'], status: '新增中', color: '#e879f9' }
      ]
    }
    const notesByScene = {
      customer: '客户经营相关的公共支撑费用按客群收入贡献比例分配至各客群成本，审批通过后生效。财务系统改造后，原始费用数据自动同步，无需人工录入。',
      product: '产品推广相关的公共支撑费用按产品线收入贡献比例分配至各产品，审批通过后生效。财务系统改造后，原始费用数据自动同步，无需人工录入。',
      channel: '渠道运营公共费用按网点数量、面积、业务量等维度分配至各渠道，数据从资产管理系统和核心业务系统自动拉取，审批通过后生效。',
      public: '公共管理费用从财务系统原始台账自动接入，按总行→分行→条线逐层分配，分摊因子从核心业务、人力资源、资产管理系统自动获取，审批流确认后生效。'
    }
    return {
      title: `${sceneLabel} - 外部系统数据接入`,
      systems: systemsByScene[scene] || systemsByScene.customer,
      note: notesByScene[scene] || notesByScene.customer
    }
  }
})

function clearLogs() {
  integrationLogs.value = []
}

// ============================================================
// 自动同步：当场景或方法切换时，更新表单数据
// ============================================================
function applySceneData() {
  const s = props.scenes[props.activeScene]
  const m = props.activeMethod

  if (m === 'direct') {
    // 直接归集：客群/产品/渠道/活动/项目维度
    const docMap = { customer: '86', product: '87', channel: '88', public: '89' }
    const eventMap = { customer: 'ACT-PRIV-2026Q1-015', product: 'PRD-MKT-2026-006', channel: 'CHL-OP-2026-001', public: 'PUB-MGMT-2026-001' }
    Object.assign(props.collectForm, {
      documentNo: 'FY-2026-0419-00' + docMap[props.activeScene],
      costSubject: s.costType,
      amount: s.amount,
      org: '九江银行南昌分行',
      line: '零售金融部',
      channel: s.channel.split('/')[0].trim(),
      productCode: s.product.split('/')[0].trim(),
      customerSegment: s.customer,
      eventCode: eventMap[props.activeScene],
      owner: s.manager,
      isDirect: '是（直连经营对象）',
      notes: `${s.label}成本归集，归集方法：${s.method}`
    })
  } else if (m === 'service') {
    // 服务关系归集：渠道运营 → 工时分配至渠道/机构，而非客户
    if (props.activeScene === 'channel') {
      Object.assign(props.collectForm, {
        documentNo: 'GS-2026-0419-0023',
        serverStaff: '渠道运营部 / 各渠道负责人',
        serviceTargetType: '渠道',
        serviceTargetCode: '营业网点 / 手机银行 / 企业网银 / 客服中心',
        serviceAction: `针对手机银行/网银/网点/客服各渠道的运营支撑服务，记录服务时长及工时分摊`,
        servicePeriod: '2026-04-01 至 2026-04-30',
        workHours: 24,
        workHourRatio: 18,
        benefitProducts: '全产品线',
        benefitOrgs: '九江银行全辖渠道',
        enterPool: '直接归集',
        costSubject: '渠道运营费（设备/人力/场地/通信）',
        driverType: '工时占比',
        driverValue: 24,
        ratio: 18,
        dataSource: 'CRM系统（改造）',
        collectFreq: '月',
        poolLevel: '',
        benefitScope: '',
        poolDimension: '',
        poolRuleCode: '',
        trialAmount: 0,
        approvalStatus: ''
      })
    } else if (props.activeScene === 'public') {
      Object.assign(props.collectForm, {
        documentNo: 'GS-2026-0419-0024',
        serverStaff: '总行运营管理部 / 各分行综合部',
        serviceTargetType: '条线/机构',
        serviceTargetCode: '零售金融部 / 对公业务部 / 普惠金融部',
        serviceAction: `针对各业务条线的公共管理支撑服务，记录支撑工时及分摊比例`,
        servicePeriod: '2026-04-01 至 2026-04-30',
        workHours: 16,
        workHourRatio: 10,
        benefitProducts: '全产品线',
        benefitOrgs: '总行/南昌分行/九江分行/福州分行',
        enterPool: '进入服务成本池',
        costSubject: '公共管理费（人力/场地/合规/品牌）',
        driverType: '工时占比',
        driverValue: 16,
        ratio: 10,
        dataSource: '人力资源系统（改造）',
        collectFreq: '月',
        poolLevel: '',
        benefitScope: '',
        poolDimension: '',
        poolRuleCode: '',
        trialAmount: 0,
        approvalStatus: ''
      })
    } else {
      // 客户/产品场景的服务关系归集
      const hourMap = { customer: 48, product: 36 }
      const ratioMap = { customer: 32, product: 25 }
      const docMap = { customer: '21', product: '22' }
      Object.assign(props.collectForm, {
        documentNo: 'GS-2026-0419-00' + docMap[props.activeScene],
        serverStaff: s.manager,
        serviceTargetType: '客户/客群',
        serviceTargetCode: s.customer,
        serviceAction: `针对${s.customer}的${s.label}服务，记录服务时长、覆盖范围及工时分摊`,
        servicePeriod: '2026-04-01 至 2026-04-30',
        workHours: hourMap[props.activeScene],
        workHourRatio: ratioMap[props.activeScene],
        benefitProducts: s.product,
        benefitOrgs: '九江银行南昌分行',
        enterPool: '直接归集',
        costSubject: s.costType,
        driverType: '工时占比',
        driverValue: hourMap[props.activeScene],
        ratio: ratioMap[props.activeScene],
        dataSource: 'CRM系统',
        collectFreq: '月',
        poolLevel: '',
        benefitScope: '',
        poolDimension: '',
        poolRuleCode: '',
        trialAmount: 0,
        approvalStatus: ''
      })
    }
  } else if (m === 'driver') {
    // 驱动因子分摊：每个场景有各自的驱动因子和受益对象类型
    const driverData = {
      customer: { item: '客户经理管户工时成本', driver: '客户数（AUM占比）', value: 328, ratio: 62, benefitType: '客群', benefitCode: 'A03财富客群（高净值客户）', src: 'CRM系统' },
      product: { item: '产品推广运营费', driver: '授信转化客户数', value: 286, ratio: 55, benefitType: '产品', benefitCode: '普惠经营贷 / 稳利盈系列', src: '核心业务系统' },
      channel: { item: '自助设备运营费', driver: '设备开机时长', value: 6000, ratio: 45, benefitType: '渠道', benefitCode: '手机银行 / 网上银行 / 营业网点 / 客服中心', src: 'IT监控系统（改造）' },
      public: { item: '科技系统运维费', driver: '工时占比', value: 1200, ratio: 30, benefitType: '条线', benefitCode: '零售金融部 / 对公业务部 / 普惠金融部', src: '人力资源系统（改造）' }
    }
    const docMap = { customer: '31', product: '32', channel: '33', public: '34' }
    const d = driverData[props.activeScene]
    Object.assign(props.collectForm, {
      documentNo: 'DF-2026-0419-00' + docMap[props.activeScene],
      costSubject: d.item,
      driverType: d.driver,
      driverValue: d.value,
      ratio: d.ratio,
      benefitType: d.benefitType,
      benefitCode: d.benefitCode,
      dataSource: d.src,
      collectFreq: '月',
      amount: 0, org: '', line: '', channel: '', productCode: '',
      customerSegment: '', eventCode: '', owner: '',
      workHours: 0, workHourRatio: 0,
      poolLevel: '', benefitScope: '', poolDimension: '',
      poolRuleCode: '', trialAmount: 0, approvalStatus: ''
    })
  } else {
    // 公共成本池分摊
    const poolData = {
      customer: { item: '总行公共管理费-客户经营支撑', level: '总行公共池', nature: '品牌支撑', scope: '全行客户经营条线', dimension: '按收入贡献', ruleCode: 'FR-P001', amount: 580000, status: '待审批' },
      product: { item: '分行运营支撑费-产品推广', level: '分行公共池', nature: '运营支撑', scope: '零售金融部', dimension: '按网点数量', ruleCode: 'FR-P002', amount: 320000, status: '试算中' },
      channel: { item: '条线公共费-渠道运营', level: '条线公共池', nature: '管理费用', scope: '渠道运营部', dimension: '按业务量', ruleCode: 'FR-P003', amount: 210000, status: '已批准' },
      public: { item: '网点公共运营费-综合支撑', level: '网点公共池', nature: '人力成本', scope: '全辖网点', dimension: '按面积', ruleCode: 'FR-P004', amount: 145000, status: '待分摊' }
    }
    const docMap = { customer: '41', product: '42', channel: '43', public: '44' }
    const p = poolData[props.activeScene]
    Object.assign(props.collectForm, {
      documentNo: 'CP-2026-0419-00' + docMap[props.activeScene],
      costSubject: p.item,
      poolLevel: p.level,
      poolNature: p.nature,
      benefitScope: p.scope,
      poolDimension: p.dimension,
      poolRuleCode: p.ruleCode,
      trialAmount: p.amount,
      approvalStatus: p.status,
      amount: 0, org: '', line: '', channel: '', productCode: '',
      customerSegment: '', eventCode: '', owner: '', isDirect: '',
      serverStaff: '', serviceTargetType: '', serviceTargetCode: '',
      serviceAction: '', servicePeriod: '', workHours: 0, workHourRatio: 0,
      benefitProducts: '', benefitOrgs: '', enterPool: '',
      driverType: '', driverValue: 0, ratio: 0, dataSource: '', collectFreq: '',
      benefitType: '', benefitCode: '',
      notes: `成本池层级：${p.level}，分摊维度：${p.dimension}，分摊规则：${p.ruleCode}，试算金额：¥${p.amount.toLocaleString()}`
    })
  }
}

// 场景或方法切换时，自动更新表单数据
watch(
  [() => props.activeScene, () => props.activeMethod],
  () => {
    applySceneData()
  },
  { immediate: true }
)
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

    <!-- ==================== 数据接入来源面板 ==================== -->
    <article class="card panel data-source-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">数据接入</p>
          <h3>外部系统对接来源</h3>
        </div>
        <span class="mini">本方法涉及的数据系统</span>
      </div>

      <!-- 系统接入卡片 -->
      <div class="source-systems">
        <div
          v-for="sys in integrationSources.systems"
          :key="sys.name"
          class="source-card"
          :style="{ '--src-color': sys.color }"
        >
          <div class="source-card-head">
            <strong>{{ sys.name }}</strong>
            <span class="source-status" :class="'status-' + sys.status">{{ sys.status }}</span>
          </div>
          <p class="source-role">{{ sys.role }}</p>
          <div class="source-fields">
            <span v-for="f in sys.fields" :key="f" class="field-badge">{{ f }}</span>
          </div>
        </div>
      </div>

      <!-- 接入说明 -->
      <div class="source-note">
        <span class="note-icon">ℹ</span>
        <p>{{ integrationSources.note }}</p>
      </div>

      <!-- 数据加载日志 -->
      <div class="integration-log">
        <div class="log-header">
          <strong>数据接入日志</strong>
          <div class="log-controls">
            <div v-if="isLoading" class="progress-bar">
              <div class="progress-fill" :style="{ width: loadProgress + '%' }"></div>
              <span class="progress-text">{{ loadProgress }}%</span>
            </div>
            <button v-if="!isLoading" class="btn-clear" @click="clearLogs">清空</button>
            <span v-if="isLoading" class="loading-dot">
              <span></span><span></span><span></span>
            </span>
          </div>
        </div>
        <div class="log-entries" v-if="integrationLogs.length > 0">
          <div
            v-for="(log, i) in integrationLogs"
            :key="i"
            class="log-entry"
            :class="'log-' + log.type"
          >
            <span class="log-time">{{ log.time }}</span>
            <span class="log-system">{{ log.system }}</span>
            <span class="log-icon">
              <template v-if="log.type === 'api'">↗</template>
              <template v-else-if="log.type === 'calc'">⚙</template>
              <template v-else-if="log.type === 'check'">✓</template>
              <template v-else-if="log.type === 'done'">✓</template>
            </span>
            <span class="log-msg">{{ log.message }}</span>
          </div>
        </div>
        <div v-else class="log-empty">
          点击「加载示例数据」查看外部系统数据接入过程
        </div>
      </div>
    </article>

    <!-- ==================== 业务录入表单 ==================== -->
    <article class="card panel form-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">{{ getMethodPage(activeMethod).system }}</p>
          <h3>{{ getMethodPage(activeMethod).name }}</h3>
        </div>
        <button class="btn-apply" :class="{ loading: isLoading }" @click="handleApply" :disabled="isLoading">
          <span v-if="!isLoading">加载示例数据</span>
          <span v-else>接入中...</span>
        </button>
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
            <div class="driver-target driver-target-head">
              <span>受益对象</span>
              <span>分摊比例</span>
              <span>驱动值</span>
              <strong>分摊金额</strong>
            </div>
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

/* ======================================================
   数据接入来源面板
   ====================================================== */
.data-source-panel {
  grid-column: 1 / -1;
}

.source-systems {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
  margin-bottom: 12px;
}

.source-card {
  padding: 12px;
  border-radius: 12px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  border-top: 2px solid var(--src-color);
}

.source-card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}
.source-card-head strong { font-size: 12px; color: var(--src-color); }

.source-status {
  font-size: 10px;
  padding: 2px 8px;
  border-radius: 999px;
}
.status-已对接 { background: rgba(73,220,177,.2); color: var(--teal); }
.status-新增中 { background: rgba(247,196,106,.2); color: var(--gold); }
.status-改造中 { background: rgba(85,200,255,.2); color: var(--cyan); }

.source-role { font-size: 11px; color: var(--muted); margin-bottom: 8px; line-height: 1.4; }

.source-fields { display: flex; flex-wrap: wrap; gap: 4px; }
.field-badge {
  font-size: 10px;
  padding: 2px 6px;
  border-radius: 4px;
  background: rgba(255,255,255,.06);
  color: var(--muted);
  font-family: monospace;
}

.source-note {
  display: flex;
  gap: 8px;
  padding: 10px 12px;
  border-radius: 10px;
  background: rgba(85,200,255,.04);
  border: 1px solid var(--line);
  margin-bottom: 12px;
}
.note-icon { color: var(--cyan); flex-shrink: 0; }
.source-note p { font-size: 12px; color: var(--muted); margin: 0; line-height: 1.5; }

/* 接入日志 */
.integration-log {
  border: 1px solid var(--line-strong);
  border-radius: 12px;
  overflow: hidden;
}

.log-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 14px;
  background: rgba(255,255,255,.03);
  border-bottom: 1px solid var(--line);
}
.log-header strong { font-size: 12px; }

.log-controls { display: flex; align-items: center; gap: 10px; }

.progress-bar {
  position: relative;
  width: 80px;
  height: 18px;
  border-radius: 999px;
  background: rgba(255,255,255,.06);
  overflow: hidden;
}
.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, var(--teal), var(--cyan));
  border-radius: 999px;
  transition: width .3s ease;
}
.progress-text {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  color: white;
  font-weight: bold;
}

.loading-dot { display: flex; gap: 4px; align-items: center; }
.loading-dot span {
  width: 5px; height: 5px; border-radius: 50%;
  background: var(--cyan);
  animation: dot-pulse 1.2s ease-in-out infinite;
}
.loading-dot span:nth-child(2) { animation-delay: .2s; }
.loading-dot span:nth-child(3) { animation-delay: .4s; }
@keyframes dot-pulse {
  0%, 80%, 100% { opacity: .3; transform: scale(.8); }
  40% { opacity: 1; transform: scale(1); }
}

.btn-clear {
  font-size: 11px;
  padding: 3px 10px;
  border-radius: 6px;
  border: 1px solid var(--line-strong);
  background: rgba(255,255,255,.04);
  color: var(--muted);
  cursor: pointer;
}
.btn-clear:hover { background: rgba(255,255,255,.08); }

.log-entries {
  max-height: 200px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: rgba(85,200,255,.3) transparent;
}

.log-entry {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 7px 14px;
  border-bottom: 1px solid rgba(255,255,255,.02);
  font-size: 11px;
  font-family: 'Fira Code', 'Consolas', monospace;
  animation: log-slide-in .2s ease;
}
@keyframes log-slide-in {
  from { opacity: 0; transform: translateY(-4px); }
  to { opacity: 1; transform: translateY(0); }
}

.log-time { color: var(--muted); flex-shrink: 0; min-width: 68px; }
.log-system { color: var(--cyan); flex-shrink: 0; min-width: 90px; font-size: 10px; }
.log-icon { flex-shrink: 0; width: 14px; text-align: center; }
.log-api .log-icon { color: #55c8ff; }
.log-calc .log-icon { color: var(--gold); }
.log-check .log-icon { color: var(--teal); }
.log-done .log-icon { color: var(--teal); }
.log-msg { color: var(--text); opacity: .85; flex: 1; line-height: 1.4; }

.log-empty {
  padding: 20px;
  text-align: center;
  color: var(--muted);
  font-size: 12px;
}

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
.btn-apply:hover:not(:disabled) { background: rgba(73,220,177,.2); }
.btn-apply:disabled { opacity: .6; cursor: not-allowed; }
.btn-apply.loading { background: rgba(85,200,255,.12); border-color: var(--cyan); color: var(--cyan); }

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
.driver-target-head { background: rgba(247,196,106,.06); font-size: 11px; color: var(--muted); }
.driver-target-head span { font-weight: bold; }
.driver-target strong { text-align: right; color: var(--gold); }

@media (max-width: 1100px) {
  .scene-tabs { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .method-tabs { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .form-grid-3 { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .driver-factor-display { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .pool-levels { grid-template-columns: 1fr; }
  .source-systems { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}
</style>
