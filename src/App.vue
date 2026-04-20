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
    amount: 186000,
    method: '第一分法（直连）+ 第二分法（服务关系）',
    methodDetail: {
      direct: { rule: 'FR-D001 客户直归法', ratio: 35, amount: 65100, target: 'A03财富客群' },
      service: { rule: 'FR-S001 客户贡献度归集法', ratio: 38, amount: 70680, target: '高净值客户管户' },
      driver: { rule: 'FR-V003 客户活跃度分摊法', ratio: 18, amount: 33480, target: '活跃客户数' },
      pool: { rule: '公共成本池', ratio: 9, amount: 16740, target: '总行管理支撑' }
    },
    // 数据接入：显性费用走费用报销，隐性工时走CRM/HR，驱动因子走核心系统
    dataIngestion: {
      primarySource: '费用报销系统（显性费用：礼品/权益/沙龙）',
      serviceSource: 'CRM系统（管户工时） + 人力资源系统（岗位条线映射）',
      driverSource: '核心业务系统（活跃客户数/交易量）',
      note: '客户沙龙礼品等显性费用在费用报销系统录入客群编码自动归集；客户经理管户工时由CRM系统按AUM占比自动计算归集，无需人工录入。'
    },
    businessFlow: [
      { step: '费用申请', action: '费用报销系统：录入客群编码（A03）、产品编码、渠道标识，是否直归标记', system: '费用报销系统' },
      { step: '工时自动归集', action: 'CRM系统：客户经理张岚月均管户工时[48h]，AUM≥600万客群AUM占比[62%]，工时成本自动写入归集台账', system: 'CRM系统（改造）' },
      { step: '权益发放归集', action: '权益平台（改造）：客户权益领取记录按客群编码自动归集至A03客群，无需人工录入', system: '权益平台（改造）' },
      { step: '驱动因子匹配', action: '管理会计平台：按规则[FR-V003]命中活跃客户数驱动因子，从核心系统拉取活跃客户数[328户]，自动分摊', system: '管理会计平台' }
    ],
    roi: { revenue: 357000, cost: 186000, roi: 1.92, suggestion: '加大AUM≥600万客群投入，ROI已达1.92' }
  },
  product: {
    label: '产品销售推动',
    fourLabel: '产品销售成本',
    customer: '小微企业客群 / 新客群',
    product: '普惠经营贷 / 稳利盈系列',
    channel: '企业网银 / 客户经理直销',
    manager: '产品经理李峻 / 客户经理王强',
    costType: '产品推广费（广告投放/佣金/激励）',
    amount: 528000,
    method: '第一分法（直连）+ 第三分法（驱动因子）',
    methodDetail: {
      direct: { rule: 'FR-D002 产品直归法', ratio: 30, amount: 158400, target: '普惠经营贷' },
      service: { rule: 'FR-S002 产品经理工时归集法', ratio: 22, amount: 116160, target: '产品经理支持' },
      driver: { rule: 'FR-V001 授信转化分摊法', ratio: 35, amount: 184800, target: '授信转化客户数' },
      pool: { rule: '公共成本池', ratio: 13, amount: 68640, target: '产品管理支撑' }
    },
    // 数据接入：推广费从营销平台直采，授信转化从核心系统拉取
    dataIngestion: {
      primarySource: '营销管理平台（推广计划/广告投放/佣金激励）',
      serviceSource: 'CRM系统（产品经理支持工时） + 产品管理系统（产品-条线映射）',
      driverSource: '核心业务系统（授信转化客户数） + CRM系统（转化线索）',
      note: '产品推广费在营销管理平台登记推广计划，转化线索从数字营销平台API自动拉取；授信转化客户数从核心业务系统定期同步，无需人工填报。'
    },
    businessFlow: [
      { step: '推广计划登记', action: '营销管理平台：录入推广产品编码、渠道、目标客群、预算金额，自动关联费用申请', system: '营销管理平台' },
      { step: '转化线索归集', action: '数字营销平台（改造）：广告投放转化数据从API自动拉取，授信转化客户数[286户]同步至核心系统', system: '数字营销平台（改造）' },
      { step: '佣金激励发放', action: '费用报销系统：录入受益产品编码、客户经理、产品编号，审批通过后直接归集', system: '费用报销系统' },
      { step: '驱动因子匹配', action: '管理会计平台：按规则[FR-V001]授信转化分摊法，转化客户数自动分配至各受益产品', system: '管理会计平台' }
    ],
    roi: { revenue: 814000, cost: 528000, roi: 1.54, suggestion: '企业网银渠道ROI偏低，建议优化渠道配置' }
  },
  channel: {
    label: '渠道运营支撑',
    fourLabel: '渠道服务成本',
    customer: '全量客户（按渠道分层）',
    product: '全产品线',
    channel: '手机银行 / 网上银行 / 营业网点 / 客服中心',
    manager: '渠道运营部 / 各渠道负责人',
    costType: '渠道运营费（设备/人力/场地/通信）',
    amount: 342000,
    method: '第三分法（驱动因子）+ 第四分法（公共池）',
    methodDetail: {
      direct: { rule: 'FR-D003 渠道专属成本直归', ratio: 18, amount: 61560, target: '网点专属设备' },
      service: { rule: 'FR-S003 渠道服务关系归集法', ratio: 25, amount: 85500, target: '渠道服务工时' },
      driver: { rule: 'FR-V002 渠道交易量分摊法', ratio: 42, amount: 143640, target: '交易笔数/金额' },
      pool: { rule: '公共成本池', ratio: 15, amount: 51300, target: '渠道公共支撑' }
    },
    // 数据接入：主要从核心/IT监控/资产系统自动拉取，无需费用报销
    dataIngestion: {
      primarySource: '财务系统（成本原始发生额） + 核心业务系统（交易量）',
      serviceSource: 'CRM系统（渠道运营人员工时） + 人力资源系统（岗位条线映射）',
      driverSource: '核心业务系统（交易笔数/金额） + IT监控系统（设备开机时长） + 资产管理系统（网点面积）',
      note: '渠道运营成本主要通过业务系统自动接入：交易量从核心业务系统拉取、设备台账从IT监控平台同步、网点面积从资产管理系统接入，按驱动因子自动分摊，无需在费用报销系统中逐笔录入。'
    },
    businessFlow: [
      { step: '交易量自动拉取', action: '核心业务系统（改造）：各渠道交易笔数/金额按月自动同步至管理会计平台，渠道标识[手机银行/网银/网点/客服]', system: '核心业务系统（改造）' },
      { step: '设备台账接入', action: 'IT监控系统（改造）：ATM/CDM设备开机时长[6,000台时]、系统调用量[128万次]自动同步，网点设备台账同步', system: 'IT监控系统（改造）' },
      { step: '网点资源数据', action: '资产管理系统（改造）：网点面积[12,800㎡]、设备台数[128台]按月同步，支持面积因子分摊', system: '资产管理系统（改造）' },
      { step: '驱动因子匹配', action: '管理会计平台：按规则[FR-V002]渠道交易量分摊法，按各渠道实际交易量占比自动计算分摊结果，无需人工干预', system: '管理会计平台' }
    ],
    roi: { revenue: 791000, cost: 342000, roi: 2.31, suggestion: '手机银行渠道ROI最优，建议加大数字化投入' }
  },
  public: {
    label: '公共管理支撑',
    fourLabel: '公共支撑成本',
    customer: '全行公共',
    product: '全产品线（受益分摊）',
    channel: '全渠道',
    manager: '总行运营管理部 / 各分行综合部',
    costType: '公共管理费（人力/场地/合规/品牌）',
    amount: 1150000,
    method: '第四分法（公共成本池分层分摊）',
    methodDetail: {
      direct: { rule: '公共成本直归', ratio: 5, amount: 57500, target: '可直接归属的公共费' },
      service: { rule: 'FR-S004 管理支撑服务归集法', ratio: 18, amount: 207000, target: '各条线管理支撑' },
      driver: { rule: 'FR-V004 公共资源驱动分摊法', ratio: 32, amount: 368000, target: '面积/人数/收入' },
      pool: { rule: '公共成本池（分层）', ratio: 45, amount: 517500, target: '总行→分行→条线逐级分配' }
    },
    // 数据接入：公共费用从财务系统直采，分摊因子从多系统拉取
    dataIngestion: {
      primarySource: '财务系统（公共费用原始台账直采，按成本池层级自动分类）',
      serviceSource: '人力资源系统（岗位-条线映射、工时数据） + 核心业务系统（收入贡献）',
      driverSource: '核心业务系统（收入贡献） + 人力资源系统（人数） + 资产管理系统（面积）',
      note: '公共管理费用从财务系统原始台账自动接入，无需人工录入；受益范围识别与分摊因子从核心业务、人力资源、资产管理系统自动拉取，按分层规则逐级分摊。'
    },
    businessFlow: [
      { step: '公共费用直采', action: '财务系统（改造）：总行/分行公共管理费用[1,150,000元]按成本池层级[总行/分行/条线/网点]自动分类，实时同步至管理会计平台', system: '财务系统（改造）' },
      { step: '受益范围识别', action: '核心业务系统（改造）：各分行收入贡献比例[Nanchang:32%/Jiujiang:28%/Fuzhou:40%]自动计算，受益条线识别', system: '核心业务系统（改造）' },
      { step: '分摊因子获取', action: '人力资源系统（改造）：各条线人数[零售:1,280人/对公:860人/普惠:540人]、资产管理系统：网点面积自动同步', system: '人力资源系统（改造）' },
      { step: '分层分摊执行', action: '管理会计平台：按规则[FR-P001/FR-P002/FR-P003]执行"总行→分行→条线"逐级分摊，试算结果提交审批流', system: '管理会计平台' }
    ],
    roi: { revenue: 0, cost: 1150000, roi: 0, suggestion: '公共成本需通过经营分析间接评估价值贡献' }
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
  { step: '01 原始费用入账', amount: 186000, desc: '客户沙龙活动及礼遇费用原始发生', rule: '无（原始入账）' },
  { step: '02 直接归集识别', amount: 65100, desc: '可直连A03财富客群的高净值礼品费', rule: 'FR-D001 客户直归法' },
  { step: '03 服务关系归集', amount: 70680, desc: '客户经理张岚管户工时归集至A03客群', rule: 'FR-S001 客户贡献度归集法' },
  { step: '04 驱动因子分摊', amount: 33480, desc: '按活跃客户数将运营看板费用分摊至客群', rule: 'FR-V003 客户活跃度分摊法' },
  { step: '05 公共池分配', amount: 16740, desc: '管理支撑费用按比例分配至客群成本', rule: 'FR-P001 总行管理费分配规则' }
]

const traceRows = [
  ['费用入账', '客户沙龙活动', '原始金额', 186000, '费用报销系统', '已入账'],
  ['直归判定', 'A03财富客群', 'FR-D001', 65100, '归集台账-客户维', '已归集'],
  ['服务归集', '高净值客户管户', 'FR-S001', 70680, '归集台账-客群维', '已归集'],
  ['驱动分摊', '活跃客户数驱动', 'FR-V003', 33480, '归集台账-驱动维', '已分摊'],
  ['公共池分配', '总行管理费池', 'FR-P001', 16740, '归集台账-公共池', '待审批']
]

// ============================================================
// ROI分析数据
// ============================================================
const productRows = [
  { name: '个人储蓄存款（定期）', type: '储蓄存款', clients: 2860, revenue: 428000, cost: 186000, roi: 2.30, status: '投入产出优', suggestion: '维持定期存款产品投入，加大三年期以上营销' },
  { name: '普惠小微贷款', type: '普惠贷款', clients: 842, revenue: 814000, cost: 528000, roi: 1.54, status: '渠道待优化', suggestion: '优化贷款审批流程，提升小企业主获客效率' },
  { name: '个人住房贷款', type: '个人贷款', clients: 1246, revenue: 386000, cost: 245000, roi: 1.58, status: '良好', suggestion: '稳定投放规模，关注不良率控制' },
  { name: '手机银行', type: '数字渠道', clients: 342000, revenue: 386000, cost: 68000, roi: 5.68, status: 'ROI最优', suggestion: '继续加大数字化投入，扩大手机银行渗透率' },
  { name: '理财代销（稳利盈系列）', type: '中间业务', clients: 892, revenue: 357000, cost: 186000, roi: 1.92, status: '投入产出优', suggestion: '扩大代销理财规模，筛选优质产品' }
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
  { system: '信贷管理系统', priority: '中', changes: ['新增信贷产品与条线映射', '新增贷款余额/发放额/户数字段', '新增不良率关联成本归集'], phase: '建设期' },
  { system: '财务系统', priority: '高', changes: ['新增公共费用原始台账接口', '新增成本科目标准化配置', '新增分摊规则审批状态回写'], phase: '试点期前完成' },
  { system: '核心业务系统', priority: '中', changes: ['新增交易量按渠道分类统计', '新增产品-机构归属映射', '新增客户-产品-渠道交叉标识', '新增系统调用量日志沉淀'], phase: '建设期' },
  { system: 'IT监控与日志平台', priority: '中', changes: ['新增系统调用量台账', '新增设备使用量采集', '新增线上渠道行为日志沉淀'], phase: '建设期' },
  { system: '资产管理系统', priority: '中', changes: ['新增网点面积与设备台账接口', '新增场地占用分摊因子配置', '新增资产折旧成本归集接口'], phase: '建设期' },
  { system: '产品管理系统', priority: '高', changes: ['新增产品目录标准化配置', '新增产品-条线映射管理', '新增产品生命周期成本归集'], phase: '试点期前完成' },
  { system: '信息科技项目管理系统', priority: '中', changes: ['新增信息系统开发成本归集', '新增项目工时与资产关联', '新增科技投入产出分析接口'], phase: '建设期' },
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

// 时间周期管理
const periodOptions = [
  { label: '日自动化（T+1）', value: 'daily-auto', start: '', end: '', desc: '每日自动跑批，结果由用户确认生效', mode: 'auto', status: 'running' },
  { label: '月度（自动）', value: 'monthly-auto', start: '', end: '', desc: '每月末日自动归集，次月初用户确认', mode: 'auto', status: 'idle' },
  { label: '季度（手工试算）', value: 'quarterly-trial', start: '2026-01-01', end: '2026-03-31', desc: '季度手工试算，用于校验自动化结果', mode: 'manual', status: 'idle' },
  { label: '2026年一季度', value: '2026Q1', start: '2026-01-01', end: '2026-03-31', desc: '2026年Q1历史数据', mode: 'history', status: 'confirmed' },
  { label: '2025年四季度', value: '2025Q4', start: '2025-10-01', end: '2025-12-31', desc: '2025年Q4历史数据', mode: 'history', status: 'confirmed' },
  { label: '2025年三季度', value: '2025Q3', start: '2025-07-01', end: '2025-09-30', desc: '2025年Q3历史数据', mode: 'history', status: 'confirmed' },
]
const activePeriod = ref('daily-auto')
const showPeriodModal = ref(false)
const selectedPeriodOption = computed(() => periodOptions.find(p => p.value === activePeriod.value) || periodOptions[0])
const periodRange = computed(() => {
  const p = selectedPeriodOption.value
  if (p.mode === 'auto') return p.desc
  return p.start && p.end ? `${p.start} ~ ${p.end}` : p.desc
})

function openPeriodModal() { showPeriodModal.value = true }
function selectPeriod(val) {
  activePeriod.value = val
  showPeriodModal.value = false
}

// 自动化跑批模拟状态
const autoRunStatus = ref('idle') // idle | running | pending | confirmed | error
const autoRunProgress = ref(0)
const autoRunLogs = ref([])
const autoRunDate = ref('')

function startAutoRun() {
  if (autoRunStatus.value === 'running') return
  autoRunStatus.value = 'running'
  autoRunProgress.value = 0
  autoRunLogs.value = []
  autoRunDate.value = new Date().toLocaleDateString('zh-CN')

  const steps = [
    { label: '拉取昨日费用报销数据', duration: 600, icon: '↗' },
    { label: '拉取CRM工时登记数据', duration: 500, icon: '↗' },
    { label: '拉取核心系统交易量', duration: 700, icon: '↗' },
    { label: '拉取财务系统公共费用', duration: 500, icon: '↗' },
    { label: '执行直归规则 FR-D001~D004', duration: 400, icon: '⚙' },
    { label: '执行服务关系规则 FR-S001~S004', duration: 400, icon: '⚙' },
    { label: '执行驱动因子规则 FR-V001~V004', duration: 500, icon: '⚙' },
    { label: '执行公共池分摊规则 FR-P001~P004', duration: 400, icon: '⚙' },
    { label: '差异校验与预警', duration: 300, icon: '✓' },
    { label: '生成归集结果，待用户确认', duration: 200, icon: '✓' },
  ]

  let delay = 0
  steps.forEach((step, i) => {
    setTimeout(() => {
      autoRunLogs.value.unshift({ icon: step.icon, label: step.label, time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit', second: '2-digit' }) })
      autoRunProgress.value = Math.round(((i + 1) / steps.length) * 100)
      if (i === steps.length - 1) {
        setTimeout(() => {
          autoRunStatus.value = 'pending'
          autoRunProgress.value = 100
        }, 300)
      }
    }, delay)
    delay += step.duration
  })
}

function confirmAutoRun() {
  autoRunStatus.value = 'confirmed'
}

function rejectAutoRun() {
  autoRunStatus.value = 'error'
  autoRunLogs.value.unshift({ icon: '✕', label: '结果已驳回，请检查异常数据后重新归集', time: new Date().toLocaleTimeString('zh-CN', { hour: '2-digit', minute: '2-digit', second: '2-digit' }) })
}

function resetAutoRun() {
  autoRunStatus.value = 'idle'
  autoRunProgress.value = 0
  autoRunLogs.value = []
}

// 各场景数据按周期变化
const periodData = {
  '2026Q1': {
    customer: { amount: 186000, roi: 1.92, methodDetail: { direct: { ratio: 35, amount: 65100 }, service: { ratio: 38, amount: 70680 }, driver: { ratio: 18, amount: 33480 }, pool: { ratio: 9, amount: 16740 } } },
    product: { amount: 528000, roi: 1.54, methodDetail: { direct: { ratio: 30, amount: 158400 }, service: { ratio: 22, amount: 116160 }, driver: { ratio: 35, amount: 184800 }, pool: { ratio: 13, amount: 68640 } } },
    channel: { amount: 342000, roi: 2.31, methodDetail: { direct: { ratio: 18, amount: 61560 }, service: { ratio: 25, amount: 85500 }, driver: { ratio: 42, amount: 143640 }, pool: { ratio: 15, amount: 51300 } } },
    public: { amount: 1150000, roi: 0, methodDetail: { direct: { ratio: 5, amount: 57500 }, service: { ratio: 18, amount: 207000 }, driver: { ratio: 32, amount: 368000 }, pool: { ratio: 45, amount: 517500 } } }
  },
  '2025Q4': {
    customer: { amount: 158000, roi: 1.84, methodDetail: { direct: { ratio: 34, amount: 53720 }, service: { ratio: 37, amount: 58460 }, driver: { ratio: 19, amount: 30020 }, pool: { ratio: 10, amount: 15800 } } },
    product: { amount: 462000, roi: 1.48, methodDetail: { direct: { ratio: 29, amount: 133980 }, service: { ratio: 23, amount: 106260 }, driver: { ratio: 34, amount: 157080 }, pool: { ratio: 14, amount: 64680 } } },
    channel: { amount: 298000, roi: 2.18, methodDetail: { direct: { ratio: 17, amount: 50660 }, service: { ratio: 24, amount: 71520 }, driver: { ratio: 43, amount: 128140 }, pool: { ratio: 16, amount: 47680 } } },
    public: { amount: 1030000, roi: 0, methodDetail: { direct: { ratio: 5, amount: 51500 }, service: { ratio: 17, amount: 175100 }, driver: { ratio: 33, amount: 339900 }, pool: { ratio: 45, amount: 463500 } } }
  },
  '2025Q3': {
    customer: { amount: 142000, roi: 1.76, methodDetail: { direct: { ratio: 33, amount: 46860 }, service: { ratio: 36, amount: 51120 }, driver: { ratio: 20, amount: 28400 }, pool: { ratio: 11, amount: 15620 } } },
    product: { amount: 396000, roi: 1.41, methodDetail: { direct: { ratio: 28, amount: 110880 }, service: { ratio: 24, amount: 95040 }, driver: { ratio: 34, amount: 134640 }, pool: { ratio: 14, amount: 55440 } } },
    channel: { amount: 276000, roi: 2.05, methodDetail: { direct: { ratio: 16, amount: 44160 }, service: { ratio: 23, amount: 63480 }, driver: { ratio: 44, amount: 121440 }, pool: { ratio: 17, amount: 46920 } } },
    public: { amount: 926000, roi: 0, methodDetail: { direct: { ratio: 6, amount: 55560 }, service: { ratio: 16, amount: 148160 }, driver: { ratio: 33, amount: 305580 }, pool: { ratio: 45, amount: 416700 } } }
  },
  '2025H1': {
    customer: { amount: 468000, roi: 1.88, methodDetail: { direct: { ratio: 34, amount: 159120 }, service: { ratio: 37, amount: 173160 }, driver: { ratio: 19, amount: 88920 }, pool: { ratio: 10, amount: 46800 } } },
    product: { amount: 718000, roi: 1.45, methodDetail: { direct: { ratio: 29, amount: 208220 }, service: { ratio: 23, amount: 165140 }, driver: { ratio: 34, amount: 244120 }, pool: { ratio: 14, amount: 100520 } } },
    channel: { amount: 986000, roi: 2.12, methodDetail: { direct: { ratio: 17, amount: 167620 }, service: { ratio: 24, amount: 236640 }, driver: { ratio: 43, amount: 423980 }, pool: { ratio: 16, amount: 157760 } } },
    public: { amount: 1856000, roi: 0, methodDetail: { direct: { ratio: 5, amount: 92800 }, service: { ratio: 17, amount: 315520 }, driver: { ratio: 33, amount: 612480 }, pool: { ratio: 45, amount: 835200 } } }
  },
  '2024FY': {
    customer: { amount: 1102000, roi: 1.82, methodDetail: { direct: { ratio: 34, amount: 374680 }, service: { ratio: 37, amount: 407740 }, driver: { ratio: 19, amount: 209380 }, pool: { ratio: 10, amount: 110200 } } },
    product: { amount: 1628000, roi: 1.39, methodDetail: { direct: { ratio: 29, amount: 472120 }, service: { ratio: 23, amount: 374440 }, driver: { ratio: 34, amount: 553520 }, pool: { ratio: 14, amount: 227920 } } },
    channel: { amount: 2268000, roi: 2.05, methodDetail: { direct: { ratio: 17, amount: 385560 }, service: { ratio: 24, amount: 544320 }, driver: { ratio: 43, amount: 975240 }, pool: { ratio: 16, amount: 362880 } } },
    public: { amount: 4356000, roi: 0, methodDetail: { direct: { ratio: 5, amount: 217800 }, service: { ratio: 17, amount: 740520 }, driver: { ratio: 33, amount: 1437480 }, pool: { ratio: 45, amount: 1960200 } } }
  }
}

// 动态场景数据（随周期变化）
const dynamicScenes = computed(() => {
  const pd = periodData[activePeriod.value] || periodData['2026Q1']
  return {
    customer: { ...scenes.customer, amount: pd.customer.amount, roi: { ...scenes.customer.roi, roi: pd.customer.roi }, methodDetail: pd.customer.methodDetail },
    product: { ...scenes.product, amount: pd.product.amount, roi: { ...scenes.product.roi, roi: pd.product.roi }, methodDetail: pd.product.methodDetail },
    channel: { ...scenes.channel, amount: pd.channel.amount, roi: { ...scenes.channel.roi, roi: pd.channel.roi }, methodDetail: pd.channel.methodDetail },
    public: { ...scenes.public, amount: pd.public.amount, roi: { ...scenes.public.roi, roi: pd.public.roi }, methodDetail: pd.public.methodDetail }
  }
})

const methodForm = reactive({
  documentNo: 'FY-2026-0419-0086',
  costSubject: '客户经营费-沙龙活动',
  amount: 186000,
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
  // 直接归集
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
  isDirect: '',
  notes: '',
  // 服务关系
  serverStaff: '',
  serviceTargetType: '',
  serviceTargetCode: '',
  serviceAction: '',
  servicePeriod: '',
  workHours: 0,
  workHourRatio: 0,
  benefitProducts: '',
  benefitOrgs: '',
  enterPool: '',
  // 驱动因子
  benefitType: '',
  benefitCode: '',
  driverType: '',
  driverValue: 0,
  ratio: 0,
  dataSource: '',
  collectFreq: '',
  // 公共成本池
  poolLevel: '',
  poolNature: '',
  benefitScope: '',
  poolDimension: '',
  poolRuleCode: '',
  trialAmount: 0,
  approvalStatus: ''
})

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
        :period-label="selectedPeriodOption.label"
        :period-range="periodRange"
        @change-period="openPeriodModal"
      />

      <!-- 自动化跑批状态面板 -->
      <div v-if="autoRunStatus !== 'idle'" class="auto-run-panel" :class="'status-' + autoRunStatus">
        <div class="auto-run-header">
          <div class="auto-run-title">
            <span class="auto-icon" v-if="autoRunStatus === 'running'">&#9696;</span>
            <span class="auto-icon confirm-icon" v-else-if="autoRunStatus === 'pending'">?</span>
            <span class="auto-icon ok-icon" v-else-if="autoRunStatus === 'confirmed'">&#10003;</span>
            <span class="auto-icon err-icon" v-else-if="autoRunStatus === 'error'">&#10005;</span>
            <div>
              <strong v-if="autoRunStatus === 'running'">自动归集跑批中</strong>
              <strong v-else-if="autoRunStatus === 'pending'">自动归集完成，待您确认</strong>
              <strong v-else-if="autoRunStatus === 'confirmed'">归集结果已确认生效</strong>
              <strong v-else-if="autoRunStatus === 'error'">归集结果已驳回</strong>
              <small v-if="autoRunDate">执行日期：{{ autoRunDate }}</small>
            </div>
          </div>
          <div class="auto-run-actions">
            <!-- 进度条 -->
            <div v-if="autoRunStatus === 'running'" class="auto-progress">
              <div class="auto-progress-bar">
                <div class="auto-progress-fill" :style="{ width: autoRunProgress + '%' }"></div>
              </div>
              <span class="auto-progress-text">{{ autoRunProgress }}%</span>
            </div>
            <!-- 确认/驳回按钮 -->
            <template v-else-if="autoRunStatus === 'pending'">
              <button class="btn-reject" @click="rejectAutoRun">驳回</button>
              <button class="btn-confirm" @click="confirmAutoRun">确认归集结果</button>
            </template>
            <button v-else class="btn-reset" @click="resetAutoRun">关闭</button>
          </div>
        </div>
        <!-- 跑批日志 -->
        <div class="auto-run-logs">
          <div v-for="(log, i) in autoRunLogs" :key="i" class="auto-log-entry">
            <span class="auto-log-icon">{{ log.icon }}</span>
            <span class="auto-log-time">{{ log.time }}</span>
            <span class="auto-log-label">{{ log.label }}</span>
          </div>
        </div>
      </div>

      <!-- 仪表盘/方法总览 -->
      <DashboardPage
        v-if="activePage === 'dashboard'"
        :four-categories="fourCategories"
        :scenes="dynamicScenes"
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
        :scenes="dynamicScenes"
        :four-categories="fourCategories"
        :active-scene="activeScene"
        :active-method="activeMethod"
        :collect-form="collectForm"
        :money="money"
        @change-scene="activeScene = $event"
        @change-method="activeMethod = $event"
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
        :scenes="dynamicScenes"
        :active-scene="activeScene"
        :four-categories="fourCategories"
        :active-period="activePeriod"
        @change-scene="activeScene = $event"
      />

      <!-- ROI分析 -->
      <RoiPage
        v-else-if="activePage === 'roi'"
        :scenes="dynamicScenes"
        :active-scene="activeScene"
        :active-period="activePeriod"
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

    <!-- 时间周期选择弹窗 -->
    <Teleport to="body">
      <div v-if="showPeriodModal" class="modal-overlay" @click.self="showPeriodModal = false">
        <div class="modal-panel">
          <div class="modal-header">
            <h3>选择管理会计期间</h3>
            <button class="modal-close" @click="showPeriodModal = false">&#x2715;</button>
          </div>
          <p class="modal-desc">选择数据统计的时间范围，各周期的成本数据将自动更新</p>
          <div class="period-list">
            <button
              v-for="p in periodOptions"
              :key="p.value"
              class="period-option"
              :class="{ active: activePeriod === p.value }"
              @click="selectPeriod(p.value)"
            >
              <div class="period-option-main">
                <strong>{{ p.label }}</strong>
                <span class="period-option-range">{{ p.start }} ~ {{ p.end }}</span>
              </div>
              <span class="period-option-desc">{{ p.desc }}</span>
              <span v-if="activePeriod === p.value" class="period-option-check">&#10003;</span>
            </button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<style>
/* ======================================================
   自动化归集跑批状态面板
   ====================================================== */
.auto-run-panel {
  border-radius: 16px;
  padding: 16px 20px;
  margin: 0 24px 16px;
  border: 1px solid;
  animation: slideDown .2s ease;
}
@keyframes slideDown { from { opacity: 0; transform: translateY(-8px); } to { opacity: 1; transform: translateY(0); } }

.auto-run-panel.status-running {
  background: rgba(85,200,255,.06);
  border-color: rgba(85,200,255,.25);
}
.auto-run-panel.status-pending {
  background: rgba(247,196,106,.06);
  border-color: rgba(247,196,106,.3);
}
.auto-run-panel.status-confirmed {
  background: rgba(73,220,177,.06);
  border-color: rgba(73,220,177,.3);
}
.auto-run-panel.status-error {
  background: rgba(255,107,107,.06);
  border-color: rgba(255,107,107,.25);
}

.auto-run-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 16px;
  margin-bottom: 12px;
}

.auto-run-title {
  display: flex;
  align-items: center;
  gap: 10px;
}
.auto-icon {
  width: 32px; height: 32px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-size: 14px; flex-shrink: 0;
}
.auto-icon { background: rgba(85,200,255,.15); color: var(--cyan); animation: spin 1.5s linear infinite; }
.confirm-icon { background: rgba(247,196,106,.15); color: var(--gold); animation: none; }
.ok-icon { background: rgba(73,220,177,.15); color: var(--teal); animation: none; }
.err-icon { background: rgba(255,107,107,.15); color: #ff6b6b; animation: none; }
@keyframes spin { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }

.auto-run-title strong { display: block; font-size: 14px; color: var(--text); }
.auto-run-title small { display: block; font-size: 11px; color: var(--muted); margin-top: 2px; }

.auto-run-actions { display: flex; align-items: center; gap: 8px; flex-shrink: 0; }

/* 进度条 */
.auto-progress { display: flex; align-items: center; gap: 8px; }
.auto-progress-bar { width: 120px; height: 6px; border-radius: 999px; background: rgba(255,255,255,.08); overflow: hidden; }
.auto-progress-fill { height: 100%; background: linear-gradient(90deg, var(--cyan), var(--teal)); border-radius: 999px; transition: width .3s ease; }
.auto-progress-text { font-size: 12px; color: var(--cyan); font-weight: bold; min-width: 36px; }

/* 按钮 */
.btn-confirm {
  padding: 8px 16px;
  border-radius: 10px;
  border: none;
  background: linear-gradient(135deg, var(--teal), var(--cyan));
  color: #0a0e1a;
  font-size: 12px;
  font-weight: bold;
  cursor: pointer;
  transition: all .15s ease;
}
.btn-confirm:hover { transform: translateY(-1px); box-shadow: 0 4px 12px rgba(73,220,177,.3); }
.btn-reject {
  padding: 8px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255,107,107,.4);
  background: rgba(255,107,107,.1);
  color: #ff6b6b;
  font-size: 12px;
  cursor: pointer;
  transition: all .15s ease;
}
.btn-reject:hover { background: rgba(255,107,107,.2); }
.btn-reset {
  padding: 6px 14px;
  border-radius: 8px;
  border: 1px solid rgba(85,200,255,.2);
  background: rgba(85,200,255,.08);
  color: var(--cyan);
  font-size: 11px;
  cursor: pointer;
}
.btn-reset:hover { background: rgba(85,200,255,.15); }

/* 日志 */
.auto-run-logs {
  max-height: 180px;
  overflow-y: auto;
  scrollbar-width: thin;
  scrollbar-color: rgba(85,200,255,.2) transparent;
}
.auto-log-entry {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 5px 8px;
  border-radius: 6px;
  font-size: 11px;
  font-family: 'Fira Code', 'Consolas', monospace;
  animation: logFade .2s ease;
}
@keyframes logFade { from { opacity: 0; } to { opacity: 1; } }
.auto-log-icon { flex-shrink: 0; width: 14px; text-align: center; color: var(--cyan); }
.auto-log-time { flex-shrink: 0; color: var(--muted); min-width: 64px; }
.auto-log-label { color: var(--text); opacity: .85; }

/* 周期选择弹窗 */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.6);
  backdrop-filter: blur(4px);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  animation: fade-in .15s ease;
}
@keyframes fade-in { from { opacity: 0; } to { opacity: 1; } }

.modal-panel {
  background: #1a1d27;
  border: 1px solid rgba(85,200,255,.2);
  border-radius: 20px;
  padding: 28px;
  width: 480px;
  max-width: 90vw;
  animation: slide-up .2s ease;
}
@keyframes slide-up { from { opacity: 0; transform: translateY(16px); } to { opacity: 1; transform: translateY(0); } }

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}
.modal-header h3 { margin: 0; font-size: 16px; color: var(--cyan, #55c8ff); }
.modal-close {
  background: none;
  border: none;
  color: var(--muted, #6b7280);
  font-size: 16px;
  cursor: pointer;
  padding: 4px 8px;
}
.modal-close:hover { color: #fff; }

.modal-desc {
  color: var(--muted, #6b7280);
  font-size: 12px;
  margin: 0 0 18px;
}

.period-list { display: flex; flex-direction: column; gap: 8px; }

.period-option {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  border-radius: 12px;
  border: 1px solid rgba(85,200,255,.12);
  background: rgba(255,255,255,.02);
  cursor: pointer;
  text-align: left;
  transition: all .15s ease;
  width: 100%;
}
.period-option:hover { border-color: rgba(85,200,255,.4); background: rgba(85,200,255,.06); }
.period-option.active {
  border-color: rgba(73,220,177,.5);
  background: rgba(73,220,177,.1);
}

.period-option-main { flex: 1; }
.period-option-main strong { display: block; font-size: 14px; color: #e5e7eb; }
.period-option-range { display: block; font-size: 11px; color: #6b7280; margin-top: 2px; font-family: monospace; }
.period-option-desc { font-size: 11px; color: #6b7280; flex-shrink: 0; }
.period-option-check { color: #49dcb1; font-size: 16px; flex-shrink: 0; }
</style>
