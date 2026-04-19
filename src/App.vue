<script setup>
import { computed, reactive, ref } from 'vue'
import AppSidebar from './components/AppSidebar.vue'
import AppHero from './components/AppHero.vue'
import CostCollectionPage from './components/pages/CostCollectionPage.vue'
import DashboardPage from './components/pages/DashboardPage.vue'
import RulesPage from './components/pages/RulesPage.vue'
import ResultPage from './components/pages/ResultPage.vue'
import RoiPage from './components/pages/RoiPage.vue'
import RoadmapPage from './components/pages/RoadmapPage.vue'

// ============================================================
// 页面导航配置
// ============================================================
const navItems = [
  { id: 'dashboard', label: '方法总览', desc: '三层递进·四分法·流程改造全景' },
  { id: 'collect', label: '成本归集', desc: '业务系统改造·源头字段嵌入' },
  { id: 'rules', label: '规则引擎', desc: '场景化规则库·分摊驱动配置' },
  { id: 'result', label: '试算追踪', desc: '归集链路追溯·差异分析' },
  { id: 'roi', label: '经营ROI', desc: '多维盈利分析·投入产出回看' },
  { id: 'roadmap', label: '落地路线', desc: '试点路径·外围系统改造' }
]

// ============================================================
// 第一分法：可直连业务对象成本
// ============================================================
const directCost = {
  id: 'direct',
  label: '第一分法',
  name: '可直连业务对象成本',
  desc: '成本发生时即可明确对应某个经营对象（客户、产品、活动、项目、机构、渠道），直接归集，无需分摊',
  category: '优先直接归集',
  color: '#49dcb1',
  sourceSystem: '费用报销 / 营销管理 / 活动平台',
  businessPage: '费用申请与审批页（改造）',
  coreLogic: '在费用申请时即完成归集字段录入，审批通过后直接写入对应经营对象台账',
  fields: [
    { name: '客户号 / 客群编码', tag: 'customerCode', required: true },
    { name: '产品编码 / 产品系列', tag: 'productCode', required: true },
    { name: '活动编号 / 营销批次', tag: 'eventCode', required: true },
    { name: '项目编号', tag: 'projectCode', required: false },
    { name: '机构编码', tag: 'orgCode', required: true },
    { name: '渠道标识', tag: 'channelCode', required: true },
    { name: '成本类别', tag: 'costType', required: true },
    { name: '是否直归', tag: 'isDirect', required: true }
  ],
  typicalCosts: [
    { item: '高净值客户礼品费', target: 'A03财富客群', cost: 68000 },
    { item: '产品首发营销费', target: '稳利盈6M理财', cost: 86000 },
    { item: '沙龙活动场地费', target: '私行客户沙龙-2026Q1', cost: 42000 },
    { item: '小微客户拜访物资', target: '普惠小微客群', cost: 35000 }
  ]
}

// ============================================================
// 第二分法：可识别服务关系成本
// ============================================================
const serviceCost = {
  id: 'service',
  label: '第二分法',
  name: '可识别服务关系成本',
  desc: '成本不能直接对应单一经营对象，但可明确服务对象范围。按"谁服务谁"的关系归集人力与服务支持成本',
  category: '服务关系归集',
  color: '#55c8ff',
  sourceSystem: '人力系统 / CRM / 工时平台 / 产品管理系统',
  businessPage: '客户经理工时登记页 / 产品经理服务记录页（新增）',
  coreLogic: '先识别服务关系（服务对象、服务范围、工时占比），再按受益比例将成本分配至被服务对象',
  fields: [
    { name: '服务方岗位 / 人员', tag: 'serverStaff', required: true },
    { name: '服务对象类型', tag: 'serviceTargetType', required: true },
    { name: '服务对象编码', tag: 'serviceTargetCode', required: true },
    { name: '服务范围说明', tag: 'serviceScope', required: true },
    { name: '工时占比（%）', tag: 'workHourRatio', required: true },
    { name: '服务起止时间', tag: 'servicePeriod', required: true },
    { name: '受益机构范围', tag: 'benefitOrgs', required: false },
    { name: '受益产品范围', tag: 'benefitProducts', required: false }
  ],
  typicalCosts: [
    { item: '客户经理管户工时成本', target: '高净值客户（AUM≥600万）', service: '张岚 / 52%工时', cost: 104000 },
    { item: '产品经理支持工时成本', target: '稳利盈6M理财', service: '李峻 / 35%工时', cost: 68000 },
    { item: '运营检查支撑工时成本', target: '全辖网点', service: '运营部 / 15%工时', cost: 42000 }
  ]
}

// ============================================================
// 第三分法：可量化驱动因子成本
// ============================================================
const driverCost = {
  id: 'driver',
  label: '第三分法',
  name: '可量化驱动因子成本',
  desc: '成本本身不能直接归到对象，但其消耗可由业务量或资源量驱动。按驱动因子比例进行分摊',
  category: '驱动因子分摊',
  color: '#f7c46a',
  sourceSystem: '核心系统 / 渠道日志 / IT监控 / 资产系统 / 财务系统',
  businessPage: '驱动因子台账配置页（新增）/ 资源消耗记录页（改造）',
  coreLogic: '建立驱动因子台账（交易量、调用量、面积、设备台数等），按受益对象实际消耗量分摊成本',
  fields: [
    { name: '成本科目', tag: 'costSubject', required: true },
    { name: '受益对象类型', tag: 'benefitType', required: true },
    { name: '受益对象编码', tag: 'benefitCode', required: true },
    { name: '驱动因子类型', tag: 'driverType', required: true },
    { name: '驱动因子值（当期）', tag: 'driverValue', required: true },
    { name: '分摊比例（%）', tag: 'ratio', required: true },
    { name: '数据来源系统', tag: 'dataSource', required: true },
    { name: '采集频率', tag: 'collectFreq', required: true }
  ],
  driverTypes: [
    { type: '交易笔数', unit: '笔', example: '自助设备运营费、清分押运费' },
    { type: '交易金额', unit: '万元', example: '系统调用成本、渠道运营费' },
    { type: '客户数/活跃客户数', unit: '户', example: '客服中心运营费、柜面耗材成本' },
    { type: '设备台数/开机时长', unit: '台/小时', example: 'ATM/CDM设备运营费' },
    { type: '面积占比', unit: '%', example: '房租物业费、公共设施维护费' },
    { type: '工时占比', unit: '%', example: '科技系统运维费' }
  ],
  typicalCosts: [
    { item: '自助设备运营费（ATM/CDM）', driver: '设备开机时长', targets: [
      { name: '营业部', ratio: '38%', value: '2,280台时', amount: 68400 },
      { name: '零售支行', ratio: '34%', value: '2,040台时', amount: 61200 },
      { name: '社区银行', ratio: '28%', value: '1,680台时', amount: 50400 }
    ]},
    { item: '客服中心运营费', driver: '服务客户数', targets: [
      { name: '零售客户', ratio: '62%', value: '18.6万客户', amount: 111600 },
      { name: '公司客户', ratio: '38%', value: '11.4万客户', amount: 68400 }
    ]}
  ]
}

// ============================================================
// 第四分法：公共管理与战略支撑成本
// ============================================================
const poolCost = {
  id: 'pool',
  label: '第四分法',
  name: '公共管理与战略支撑成本',
  desc: '面向全行管理、难以直接追溯单个经营对象的成本。进入公共成本池，按分层规则逐级分配',
  category: '成本池分层分摊',
  color: '#e879f9',
  sourceSystem: '财务系统 / 管理会计平台',
  businessPage: '公共成本池管理页 / 分摊规则配置页（新增）',
  coreLogic: '先归入公共成本池，再按"总行→分行→支行/条线→产品/客户/渠道"逐层分配，有边界、有逻辑',
  fields: [
    { name: '成本池层级', tag: 'poolLevel', required: true },
    { name: '成本性质', tag: 'poolNature', required: true },
    { name: '受益范围', tag: 'benefitScope', required: true },
    { name: '分摊维度', tag: 'poolDimension', required: true },
    { name: '分摊规则编码', tag: 'poolRuleCode', required: true },
    { name: '分配比例（%）', tag: 'poolRatio', required: true },
    { name: '试算金额', tag: 'trialAmount', required: true },
    { name: '审批状态', tag: 'approvalStatus', required: true }
  ],
  poolLevels: [
    { level: '总行公共池', desc: '总行运营管理部公共费用、制度建设费用、综合培训费用', example: '总行大楼运营费、品牌支撑费' },
    { level: '分行公共池', desc: '分行运营管理费用、风险合规支持费用、公共数据治理费用', example: '分行科技支撑费、合规检查费' },
    { level: '条线公共池', desc: '各业务条线公共管理费用、条线培训费用、条线品牌支撑', example: '零售条线培训费、普惠条线推广费' },
    { level: '网点公共池', desc: '网点公共运营费用、公共设施维护费用', example: '网点公共水电费、安保费' }
  ],
  typicalCosts: [
    { item: '总行公共管理费', level: '总行公共池', amount: 580000, poolRule: '按收入贡献比例分配至各分行', status: '待分摊' },
    { item: '分行运营支撑费', level: '分行公共池', amount: 320000, poolRule: '按网点数量比例分配至各支行', status: '试算中' },
    { item: '风险合规支持费', level: '条线公共池', amount: 210000, poolRule: '按风险资产规模分配至各条线', status: '已分配' }
  ]
}

// 四分法总览数据
const fourCategories = [directCost, serviceCost, driverCost, poolCost]

// ============================================================
// 场景选择与数据
// ============================================================
const scenes = {
  customer: {
    label: '客户经营活动',
    fourLabel: '客户经营成本',
    customer: 'A03财富客群（高净值客户）',
    product: '稳利盈6M理财',
    channel: '手机银行 / 财富中心',
    manager: '客户经理张岚',
    costType: '客户经营费（礼品/权益/沙龙）',
    amount: 286000,
    method: '第一分法（直连）+ 第二分法（服务关系）',
    methodDetail: {
      direct: { rule: 'FR-D001 客户直归法', ratio: 32, amount: 91520, target: 'A03财富客群' },
      service: { rule: 'FR-S001 客户贡献度归集法', ratio: 36, amount: 102960, target: '高净值客户管户' },
      driver: { rule: 'FR-V003 客户活跃度分摊法', ratio: 22, amount: 62920, target: '活跃客户数' },
      pool: { rule: '公共成本池', ratio: 10, amount: 28600, target: '总行管理支撑' }
    },
    businessFlow: [
      { step: '费用申请', action: '在费用申请页录入：客群编码（A03）、产品编码、渠道标识、是否直归', system: '费用报销系统' },
      { step: '客户经理工时登记', action: '登记管户工时和服务对象受益范围', system: 'CRM系统（改造）' },
      { step: '权益发放记录', action: '从权益平台拉取客户领取记录，自动归集到客群', system: '权益平台（改造）' },
      { step: '沙龙活动登记', action: '录入活动编号、参会客户名单、受邀客群', system: '营销管理平台（改造）' }
    ],
    roi: { revenue: 512000, cost: 286000, roi: 1.79, suggestion: '加大AUM≥600万客群投入，ROI已达1.79' }
  },
  product: {
    label: '产品销售推动',
    fourLabel: '产品销售成本',
    customer: '小微企业客群 / 新客群',
    product: '普惠经营贷 / 稳利盈系列',
    channel: '企业网银 / 客户经理直销',
    manager: '产品经理李峻 / 客户经理王强',
    costType: '产品推广费（广告投放/佣金/激励）',
    amount: 418000,
    method: '第一分法（直连）+ 第三分法（驱动因子）',
    methodDetail: {
      direct: { rule: 'FR-D002 产品直归法', ratio: 33, amount: 137940, target: '普惠经营贷' },
      service: { rule: 'FR-S002 产品经理工时归集法', ratio: 25, amount: 104500, target: '产品经理支持' },
      driver: { rule: 'FR-V001 授信转化分摊法', ratio: 30, amount: 125400, target: '授信转化客户数' },
      pool: { rule: '公共成本池', ratio: 12, amount: 50160, target: '产品管理支撑' }
    },
    businessFlow: [
      { step: '产品推广计划申请', action: '在营销管理页录入：产品编码、推广渠道、目标客群、预算金额', system: '营销管理系统（改造）' },
      { step: '广告投放登记', action: '录入投放平台、活动编号、转化追踪码、受益产品', system: '数字营销平台（改造）' },
      { step: '转化线索归集', action: '从广告平台API拉取转化客户数、产品咨询量', system: '广告平台（对接）' },
      { step: '佣金激励发放', action: '录入受益产品、客户经理、产品编码', system: '费用报销系统（改造）' }
    ],
    roi: { revenue: 698000, cost: 418000, roi: 1.67, suggestion: '企业网银渠道ROI偏低，建议优化渠道配置' }
  },
  channel: {
    label: '渠道运营支撑',
    fourLabel: '渠道服务成本',
    customer: '全量客户（按渠道分层）',
    product: '全产品线',
    channel: '手机银行 / 网上银行 / 营业网点 / 客服中心',
    manager: '渠道运营部 / 各渠道负责人',
    costType: '渠道运营费（设备/人力/场地/通信）',
    amount: 562000,
    method: '第三分法（驱动因子）+ 第四分法（公共池）',
    methodDetail: {
      direct: { rule: 'FR-D003 渠道专属成本直归', ratio: 15, amount: 84300, target: '网点专属设备' },
      service: { rule: 'FR-S003 渠道服务关系归集法', ratio: 20, amount: 112400, target: '渠道服务工时' },
      driver: { rule: 'FR-V002 渠道交易量分摊法', ratio: 45, amount: 252900, target: '交易笔数/金额' },
      pool: { rule: '公共成本池', ratio: 20, amount: 112400, target: '渠道公共支撑' }
    },
    businessFlow: [
      { step: '渠道资源消耗登记', action: '登记各渠道设备台数、场地面积、人员工时', system: '资产管理系统（改造）' },
      { step: '交易量数据采集', action: '从核心系统拉取各渠道交易笔数、金额', system: '核心业务系统（对接）' },
      { step: '系统调用量采集', action: '从IT监控平台拉取各渠道系统调用量、负载量', system: 'IT监控系统（对接）' },
      { step: '渠道成本汇总', action: '按交易量/客户数/使用量汇总渠道分摊结果', system: '管理会计平台（改造）' }
    ],
    roi: { revenue: 1240000, cost: 562000, roi: 2.21, suggestion: '手机银行渠道ROI最优，建议加大数字化投入' }
  },
  public: {
    label: '公共管理支撑',
    fourLabel: '公共支撑成本',
    customer: '全行公共',
    product: '全产品线（受益分摊）',
    channel: '全渠道',
    manager: '总行运营管理部 / 各分行综合部',
    costType: '公共管理费（人力/场地/合规/品牌）',
    amount: 980000,
    method: '第四分法（公共成本池分层分摊）',
    methodDetail: {
      direct: { rule: '公共成本直归', ratio: 5, amount: 49000, target: '可直接归属的公共费' },
      service: { rule: 'FR-S004 管理支撑服务归集法', ratio: 15, amount: 147000, target: '各条线管理支撑' },
      driver: { rule: 'FR-V004 公共资源驱动分摊法', ratio: 30, amount: 294000, target: '面积/人数/收入' },
      pool: { rule: '公共成本池（分层）', ratio: 50, amount: 490000, target: '总行→分行→条线逐级分配' }
    },
    businessFlow: [
      { step: '公共费用归集', action: '在财务系统归集总行/分行公共管理费用', system: '财务系统' },
      { step: '成本池入池登记', action: '录入成本池层级、成本性质、受益范围', system: '管理会计平台（新增）' },
      { step: '分摊规则配置', action: '配置各层级的分摊维度、规则、受益范围', system: '管理会计平台（新增）' },
      { step: '试算与审批', action: '执行分摊试算，差异分析，提交审批', system: '管理会计平台（新增）' }
    ],
    roi: { revenue: 0, cost: 980000, roi: 0, suggestion: '公共成本需通过经营分析间接评估价值贡献' }
  }
}

// ============================================================
// 规则库
// ============================================================
const ruleGroups = [
  {
    title: '直接归集规则（第一分法）',
    code: 'FR-D',
    color: '#49dcb1',
    rules: [
      { code: 'FR-D001', name: '客户直归法', desc: '按客户号/客群编码直接归集礼品费、权益费、沙龙费等', cost: 368000 },
      { code: 'FR-D002', name: '产品直归法', desc: '按产品编码直接归集产品推广费、佣金、培训费等', cost: 286000 },
      { code: 'FR-D003', name: '活动直归法', desc: '按活动编号直接归集活动场地费、物料费、奖品费等', cost: 154000 },
      { code: 'FR-D004', name: '项目直归法', desc: '按项目编号直接归集咨询费、律师费、外部服务费等', cost: 98000 }
    ]
  },
  {
    title: '服务关系归集规则（第二分法）',
    code: 'FR-S',
    color: '#55c8ff',
    rules: [
      { code: 'FR-S001', name: '客户贡献度归集法', desc: '按客户经理管户AUM占比归集人力成本', cost: 285000 },
      { code: 'FR-S002', name: '产品经理工时归集法', desc: '按产品经理支持各产品工时占比归集人力成本', cost: 198000 },
      { code: 'FR-S003', name: '运营服务关系归集法', desc: '按运营检查服务范围归集运营支撑成本', cost: 142000 },
      { code: 'FR-S004', name: 'IT服务关系归集法', desc: '按系统服务业务范围归集科技运维成本', cost: 226000 }
    ]
  },
  {
    title: '驱动因子分摊规则（第三分法）',
    code: 'FR-V',
    color: '#f7c46a',
    rules: [
      { code: 'FR-V001', name: '授信转化分摊法', desc: '按授信转化客户数分摊产品推广成本', cost: 215000 },
      { code: 'FR-V002', name: '渠道交易量分摊法', desc: '按各渠道交易笔数/金额分摊渠道运营成本', cost: 342000 },
      { code: 'FR-V003', name: '客户活跃度分摊法', desc: '按活跃客户数分摊客服中心和客户经营成本', cost: 178000 },
      { code: 'FR-V004', name: '设备使用量分摊法', desc: '按设备开机时长/交易量分摊自助设备运营成本', cost: 96000 }
    ]
  },
  {
    title: '公共成本池分摊规则（第四分法）',
    code: 'FR-P',
    color: '#e879f9',
    rules: [
      { code: 'FR-P001', name: '总行管理费分配规则', desc: '总行→分行按收入贡献比例分配公共管理费', cost: 580000 },
      { code: 'FR-P002', name: '分行支撑费分配规则', desc: '分行→支行/网点按网点数量/面积分配支撑费', cost: 320000 },
      { code: 'FR-P003', name: '条线公共费分配规则', desc: '各条线公共费按条线收入贡献分配', cost: 210000 },
      { code: 'FR-P004', name: '网点公共费分配规则', desc: '网点公共运营费按业务量分配至各业务条线', cost: 145000 }
    ]
  }
]

// ============================================================
// 试算追踪数据
// ============================================================
const allocationSteps = [
  { step: '01 原始费用入账', amount: 286000, desc: '客户沙龙活动及礼遇费用原始发生', rule: '无（原始入账）' },
  { step: '02 直接归集识别', amount: 91520, desc: '可直连A03财富客群的高净值礼品费', rule: 'FR-D001 客户直归法' },
  { step: '03 服务关系归集', amount: 102960, desc: '客户经理张岚管户工时归集至A03客群', rule: 'FR-S001 客户贡献度归集法' },
  { step: '04 驱动因子分摊', amount: 62920, desc: '按活跃客户数将运营看板费用分摊至客群', rule: 'FR-V003 客户活跃度分摊法' },
  { step: '05 公共池分配', amount: 28600, desc: '管理支撑费用按比例分配至客群成本', rule: 'FR-P001 总行管理费分配规则' }
]

const traceRows = [
  ['费用入账', '客户沙龙活动', '原始金额', 286000, '费用报销系统', '已入账'],
  ['直归判定', 'A03财富客群', 'FR-D001', 91520, '归集台账-客户维', '已归集'],
  ['服务归集', '高净值客户管户', 'FR-S001', 102960, '归集台账-客群维', '已归集'],
  ['驱动分摊', '活跃客户数驱动', 'FR-V003', 62920, '归集台账-驱动维', '已分摊'],
  ['公共池分配', '总行管理费池', 'FR-P001', 28600, '归集台账-公共池', '待审批']
]

// ============================================================
// ROI分析数据
// ============================================================
const productRows = [
  { name: '稳利盈6M理财', type: '财富产品', clients: 126, revenue: 512000, cost: 286000, roi: 1.79, status: '投入产出优', suggestion: '维持投入规模' },
  { name: '普惠经营贷', type: '对公贷款', clients: 84, revenue: 698000, cost: 418000, roi: 1.67, status: '渠道待优化', suggestion: '优化企业网银渠道配置' },
  { name: '白鹭信用卡（新客礼遇）', type: '信用卡', clients: 214, revenue: 356000, cost: 182000, roi: 1.96, status: '获客效率优', suggestion: '加大线上获客投入' },
  { name: '手机银行运营费', type: '渠道运营', clients: 286000, revenue: 892000, cost: 312000, roi: 2.86, status: '渠道ROI最优', suggestion: '数字化投入可加大' },
  { name: '高净值客群经营', type: '客户经营', clients: 328, revenue: 1240000, cost: 586000, roi: 2.12, status: '客群价值高', suggestion: '重点加大AUM≥600万客群' }
]

const roiSummary = [
  { title: '客户视角', desc: '识别客群经营资源是否真正投入到高价值客户与高转化经营动作，支撑客群分层定价与资源差异化配置', type: '客群经营优化' },
  { title: '产品视角', desc: '比较同类产品在不同渠道、不同客群中的投入产出效率，支撑产品策略优化与产品生命周期管理', type: '产品策略优化' },
  { title: '渠道视角', desc: '分析各渠道运营成本与业务贡献，支持渠道效能评估和渠道资源配置决策', type: '渠道效能优化' },
  { title: '管理视角', desc: '将无法直归的管理支持费用纳入公共池，形成可追溯、可解释的多层分配链路，支撑条线盈利评价', type: '条线评价优化' }
]

// ============================================================
// 落地路线与系统改造
// ============================================================
const roadmap = [
  {
    phase: '咨询设计期（1-3月）',
    goal: '形成可交付蓝图',
    tasks: ['现状诊断与差距分析', '产品体系设计（含产品目录统一）', '成本归集规则设计（四分法）', '盈利分析体系设计', '系统需求与外围改造设计'],
    deliverables: ['现状诊断报告', '产品体系优化方案', '成本归集规则设计方案', '盈利分析体系设计方案', '系统功能需求说明书']
  },
  {
    phase: '试点验证期（4-6月）',
    goal: '验证关键规则',
    tasks: ['选择重点条线/产品/机构开展试点', '验证成本归集规则可行性', '验证核心盈利指标口径', '系统改造需求确认', '输出试点问题与优化建议'],
    deliverables: ['试点执行报告', '规则优化建议', '指标口径确认单', '系统改造需求确认书']
  },
  {
    phase: '建设推广期（7-12月）',
    goal: '支撑全面落地',
    tasks: ['支撑系统开发与联调测试', '规则上线与口径在全行推广', '驾驶舱与分析应用落地', '培训宣贯与治理机制固化'],
    deliverables: ['上线报告', '推广执行手册', '管理驾驶舱', '治理机制固化文档']
  }
]

const systemUpgrades = [
  { system: '费用报销系统', priority: '高', changes: ['新增客群编码字段', '新增产品编码字段', '新增活动编号字段', '新增渠道标识字段', '新增"是否直归"标记'], phase: '咨询期同步改造' },
  { system: 'CRM系统', priority: '高', changes: ['新增客户经理工时登记功能', '新增服务关系配置功能', '新增管户AUM占比自动计算', '新增服务对象受益范围登记'], phase: '试点期前完成' },
  { system: '营销管理平台', priority: '高', changes: ['新增活动编号管理', '新增营销批次管理', '新增转化线索自动归集', '新增推广产品关联配置'], phase: '试点期前完成' },
  { system: '人力资源系统', priority: '中', changes: ['新增岗位-条线关系配置', '新增工时采集接口', '新增人员-客户关系（管户）', '新增人员-产品关系（管产品）'], phase: '建设期' },
  { system: '核心业务系统', priority: '中', changes: ['新增交易量按渠道分类统计', '新增产品-机构归属映射', '新增客户-产品-渠道交叉标识', '新增系统调用量日志沉淀'], phase: '建设期' },
  { system: 'IT监控与日志平台', priority: '中', changes: ['新增系统调用量台账', '新增设备使用量采集', '新增线上渠道行为日志沉淀'], phase: '建设期' },
  { system: '管理会计平台', priority: '高', changes: ['新增规则配置引擎', '新增成本池管理功能', '新增分摊试算功能', '新增归集结果追溯', '新增多维盈利分析展示'], phase: '咨询期同步建设' }
]

// ============================================================
// 差异预警规则
// ============================================================
const issueList = [
  { title: '差异金额校验', desc: '若已归集金额与原始投入不一致，系统自动提示需调整直归比例或公共池配置', severity: 'high' },
  { title: '驱动缺失预警', desc: '当转化客户数、交易量等关键驱动因子缺失时，不允许规则直接生效，避免结果失真', severity: 'high' },
  { title: '规则版本追溯', desc: '每次试算记录所用规则版本、审批状态与数据批次，支持复盘试点问题', severity: 'medium' },
  { title: '口径一致性校验', desc: '检验产品目录、客户客群、渠道标识在多系统间的一致性', severity: 'medium' },
  { title: '归集比例合理性告警', desc: '单笔成本直归比例超过阈值（如>80%）或服务关系占比异常时告警', severity: 'low' }
]

// ============================================================
// 状态与表单
// ============================================================
const activePage = ref('dashboard')
const activeScene = ref('customer')
const activeMethod = ref('direct')

const methodForm = reactive({
  documentNo: 'FY-2026-0419-0086',
  costSubject: '客户经营费-沙龙活动',
  amount: 286000,
  org: '九江银行南昌分行',
  line: '零售金融部',
  channel: '财富中心',
  productCode: 'PRD-WL6M-2026',
  customerSegment: 'A03（高净值客户）',
  eventCode: 'ACT-PRIV-2026Q1-015',
  owner: '张岚（客户经理）',
  notes: '2026年一季度高净值客户私行沙龙活动，面向AUM≥600万客户'
})

// 成本归集表单
const collectForm = reactive({
  documentNo: '',
  costSubject: '',
  amount: 0,
  org: '',
  line: '',
  channel: '',
  productCode: '',
  customerSegment: '',
  eventCode: '',
  owner: '',
  workHours: 0,
  workHourRatio: 0,
  driverType: '',
  driverValue: 0,
  benefitType: '',
  benefitCode: '',
  poolLevel: '',
  poolRule: '',
  notes: ''
})

function applyScene() {
  const s = scenes[activeScene.value]
  methodForm.documentNo = 'FY-2026-0419-00' + (activeScene.value === 'customer' ? '86' : activeScene.value === 'product' ? '87' : activeScene.value === 'channel' ? '88' : '89')
  methodForm.costSubject = s.costType
  methodForm.amount = s.amount
  methodForm.customerSegment = s.customer
  methodForm.productCode = s.product.split('/')[0]
  methodForm.channel = s.channel
  methodForm.owner = s.manager
  methodForm.notes = `${s.label}成本归集，方法：${s.method}`
}

// 计算
const scene = computed(() => scenes[activeScene.value])
const selectedMethod = computed(() => fourCategories.find(m => m.id === activeMethod.value) || directCost)
const currentMethod = computed(() => fourCategories.find(m => m.id === activeMethod.value) || directCost)

const currentTitle = computed(() => navItems.find((item) => item.id === activePage.value)?.label ?? '成本归集演示系统')
const currentSubtitle = computed(() => {
  const subtitles = {
    dashboard: '围绕"三层递进、四分法、流程嵌入式改造"方法论，展示银行经营管理成本归集的完整链路与系统改造全景。',
    collect: '将成本归集要求嵌入业务发生源头——费用申请、作业登记、驱动采集、公共池管理四大业务场景。',
    rules: '构建场景化规则库：直接归集、服务关系、驱动因子、公共池分层四类规则，支持版本管理与试算追溯。',
    result: '展示从原始费用到多维经营对象的归集链路，支持差异分析、规则追溯、问题定位与审计回溯。',
    roi: '从客户、产品、渠道、管理四维视角回看投入产出，为资源配置和经营优化提供数据支撑。',
    roadmap: '按咨询设计—试点验证—建设推广三阶段推进，展示外围系统改造清单与治理机制安排。'
  }
  return subtitles[activePage.value] || ''
})

const money = (v) => new Intl.NumberFormat('zh-CN', {
  style: 'currency',
  currency: 'CNY',
  maximumFractionDigits: 0
}).format(v)
</script>

<template>
  <div class="app-shell">
    <AppSidebar
      :nav-items="navItems"
      :active-page="activePage"
      @navigate="activePage = $event"
    />

    <div class="content">
      <AppHero
        :title="currentTitle"
        :subtitle="currentSubtitle"
        :context-title="selectedMethod.name"
        :context-meta="selectedMethod.category"
      />

      <!-- 仪表盘/方法总览 -->
      <DashboardPage
        v-if="activePage === 'dashboard'"
        :four-categories="fourCategories"
        :scenes="scenes"
        :active-scene="activeScene"
        :active-method="activeMethod"
        :selected-method="selectedMethod"
        :money="money"
        @change-scene="activeScene = $event"
        @change-method="activeMethod = $event"
      />

      <!-- 成本归集录入 -->
      <CostCollectionPage
        v-else-if="activePage === 'collect'"
        :scenes="scenes"
        :four-categories="fourCategories"
        :active-scene="activeScene"
        :active-method="activeMethod"
        :collect-form="collectForm"
        :money="money"
        @change-scene="activeScene = $event"
        @change-method="activeMethod = $event"
        @apply-scene="applyScene"
      />

      <!-- 规则引擎 -->
      <RulesPage
        v-else-if="activePage === 'rules'"
        :rule-groups="ruleGroups"
        :four-categories="fourCategories"
        :active-method="activeMethod"
        :money="money"
        @change-method="activeMethod = $event"
      />

      <!-- 试算追踪 -->
      <ResultPage
        v-else-if="activePage === 'result'"
        :allocation-steps="allocationSteps"
        :trace-rows="traceRows"
        :issue-list="issueList"
        :money="money"
        :scenes="scenes"
        :active-scene="activeScene"
        :four-categories="fourCategories"
        @change-scene="activeScene = $event"
      />

      <!-- ROI分析 -->
      <RoiPage
        v-else-if="activePage === 'roi'"
        :product-rows="productRows"
        :roi-summary="roiSummary"
        :scenes="scenes"
        :active-scene="activeScene"
        :money="money"
        @change-scene="activeScene = $event"
      />

      <!-- 落地路线 -->
      <RoadmapPage
        v-else-if="activePage === 'roadmap'"
        :roadmap="roadmap"
        :system-upgrades="systemUpgrades"
      />
    </div>
  </div>
</template>
