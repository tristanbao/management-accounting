<script setup>
defineProps({
  fourCategories: { type: Array, required: true },
  scenes: { type: Object, required: true },
  activeScene: { type: String, required: true },
  activeMethod: { type: String, required: true },
  selectedMethod: { type: Object, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-scene', 'change-method'])

const threeTiers = [
  {
    tier: '第一层',
    name: '科目到部门',
    desc: '解决"成本看得见"问题。将成本从财务总账、费用报销、资产、人力、营销等系统归集到责任部门，建立统一成本分类体系和部门级成本台账，形成基础成本地图。',
    targets: ['运营管理部', '零售金融部', '公司金融部', '科技部', '风险管理部'],
    value: '传统财务管理向管理会计过渡的基础层',
    icon: '📊'
  },
  {
    tier: '第二层',
    name: '部门到经营对象',
    desc: '解决"成本归到谁"问题。将成本从部门口径进一步归到机构、产品、客户、渠道、条线、客群、活动、项目等经营对象。能直接识别的直接归集，不能的采集驱动因子后分摊。',
    targets: ['机构维度', '产品维度', '客户/客群维度', '渠道维度', '条线维度', '活动/项目维度'],
    value: '从"财务成本"走向"经营成本"的关键跃迁',
    icon: '🎯'
  },
  {
    tier: '第三层',
    name: '经营对象到经营动作',
    desc: '解决"成本为什么发生、是否有效"问题。将成本进一步关联到获客、激活、维护、转化、交叉销售、留存、风险管控、运营支撑、产品推广等具体经营动作。',
    targets: ['客户经营成本', '产品销售成本', '渠道服务成本', '营销ROI', '客群投入产出', '单客户边际贡献'],
    value: '行业领先：不仅知道成本归到哪个产品/客户，还知道对应哪个经营动作',
    icon: '🚀'
  }
]

const sceneSummaries = [
  { key: 'customer', label: '客户经营活动', method: '第一+第二分法', amount: 286000, color: '#49dcb1', desc: '高净值客户沙龙、权益投放、客户经理管户工时归集' },
  { key: 'product', label: '产品销售推动', method: '第一+第三分法', amount: 418000, color: '#55c8ff', desc: '产品推广费、佣金激励、渠道线索转化归集' },
  { key: 'channel', label: '渠道运营支撑', method: '第三+第四分法', amount: 562000, color: '#f7c46a', desc: '网点运营费、线上渠道、自助设备运营成本' },
  { key: 'public', label: '公共管理支撑', method: '第四分法', amount: 980000, color: '#e879f9', desc: '总行/分行公共费用、条线支撑费、网点公共池' }
]

const processSteps = [
  { phase: '业务发生', desc: '费用申请、客户经理作业、资源消耗、管理决策', color: '#49dcb1' },
  { phase: '成本识别', desc: '判断是否可直连经营对象，识别服务关系或驱动因子', color: '#55c8ff' },
  { phase: '归集判定', desc: '按四分法确定归集方法，进入对应业务流程', color: '#f7c46a' },
  { phase: '分摊驱动采集', desc: '采集工时、交易量、设备台数、面积等驱动因子', color: '#e879f9' },
  { phase: '规则引擎处理', desc: '规则匹配、金额计算、结果校验', color: '#49dcb1' },
  { phase: '多维分析输出', desc: '写入归集台账、多维盈利主题表、经营分析报表', color: '#55c8ff' }
]
</script>

<template>
  <section class="page-grid dashboard-layout">

    <!-- ==================== 方法论总览标题 ==================== -->
    <article class="card panel hero-panel">
      <div class="method-hero">
        <div class="method-title-area">
          <p class="eyebrow">银行经营管理成本归集领先方法</p>
          <h2>三层递进 · 四维四分 · 流程嵌入式改造</h2>
          <p class="method-intro">
            本方法的核心，不是简单做成本分摊，而是围绕银行管理会计落地，形成一条完整链路：
            <strong>业务发生 → 成本识别 → 归集判定 → 分摊驱动采集 → 规则引擎处理 → 多维分析输出</strong>。
            它由三部分组成：三层递进明确建设路径，四分法处理不同成本，流程嵌入式改造让归集逻辑在业务发生时就完成。
          </p>
        </div>
        <div class="process-chain">
          <div v-for="(step, idx) in processSteps" :key="idx" class="process-step">
            <div class="process-dot" :style="{ background: step.color }"></div>
            <span>{{ step.phase }}</span>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 三层递进 ==================== -->
    <article class="card panel tiers-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">方法论核心</p>
          <h3>三层递进：形成银行成本归集的演进路径</h3>
        </div>
        <span class="mini">从粗到细、从静态到动态</span>
      </div>
      <div class="tiers-grid">
        <div v-for="tier in threeTiers" :key="tier.tier" class="tier-card">
          <div class="tier-header">
            <span class="tier-icon">{{ tier.icon }}</span>
            <div>
              <strong>{{ tier.tier }}：{{ tier.name }}</strong>
              <p class="tier-value">{{ tier.value }}</p>
            </div>
          </div>
          <p class="tier-desc">{{ tier.desc }}</p>
          <div class="tier-targets">
            <span v-for="t in tier.targets" :key="t" class="tier-tag">{{ t }}</span>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 四分法概览 ==================== -->
    <article class="card panel four-cat-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">四分法</p>
          <h3>不同性质成本用不同归集方法</h3>
        </div>
        <span class="mini">成本归集核心方法</span>
      </div>
      <div class="four-cat-grid">
        <div
          v-for="cat in fourCategories"
          :key="cat.id"
          class="cat-card"
          :class="{ active: activeMethod === cat.id }"
          :style="{ '--cat-color': cat.color }"
          @click="emit('change-method', cat.id)"
        >
          <div class="cat-head">
            <strong>{{ cat.label }}</strong>
            <span class="cat-tag">{{ cat.category }}</span>
          </div>
          <h4>{{ cat.name }}</h4>
          <p class="cat-desc">{{ cat.desc }}</p>
          <div class="cat-system">
            <small>源系统：{{ cat.sourceSystem }}</small>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 典型场景汇总 ==================== -->
    <article class="card panel scenes-summary-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">典型场景</p>
          <h3>四类业务场景成本归集</h3>
        </div>
        <span class="mini">点击切换场景，查看详情</span>
      </div>
      <div class="scenes-grid">
        <div
          v-for="s in sceneSummaries"
          :key="s.key"
          class="scene-card"
          :class="{ active: activeScene === s.key }"
          :style="{ '--sc-color': s.color }"
          @click="emit('change-scene', s.key)"
        >
          <div class="scene-card-head">
            <strong>{{ s.label }}</strong>
            <span class="scene-method-tag">{{ s.method }}</span>
          </div>
          <p>{{ s.desc }}</p>
          <div class="scene-amount">
            <span>本期投入</span>
            <strong>{{ money(s.amount) }}</strong>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 四分法与三层递进对应关系 ==================== -->
    <article class="card panel relation-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">方法关联</p>
          <h3>三层递进与四分法的对应关系</h3>
        </div>
      </div>
      <div class="relation-table">
        <table>
          <thead>
            <tr>
              <th>层次</th>
              <th>对应分法</th>
              <th>典型成本</th>
              <th>归集目标</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td><span class="tier-tag">第一层</span></td>
              <td>第一类 + 第四类</td>
              <td>产品专项营销费、网点专属运营保障费、总行公共管理费</td>
              <td>先建立成本分类体系、部门台账</td>
            </tr>
            <tr>
              <td><span class="tier-tag">第二层</span></td>
              <td>第一类 + 第二类 + 第三类</td>
              <td>客户礼品费、产品经理工时、渠道交易量分摊、IT系统运维费</td>
              <td>归到客户、产品、机构、渠道等经营对象</td>
            </tr>
            <tr>
              <td><span class="tier-tag">第三层</span></td>
              <td>全部四类</td>
              <td>客户经营成本、产品销售成本、渠道服务成本、营销ROI</td>
              <td>关联经营动作，形成经营分析闭环</td>
            </tr>
          </tbody>
        </table>
      </div>
    </article>

    <!-- ==================== 当前选中方法详情 ==================== -->
    <article class="card panel method-detail-panel" :style="{ '--cat-color': selectedMethod.color }">
      <div class="title-row">
        <div>
          <p class="eyebrow">方法详情</p>
          <h3>{{ selectedMethod.name }}</h3>
        </div>
        <span class="chip" :style="{ background: selectedMethod.color + '22', color: selectedMethod.color }">{{ selectedMethod.category }}</span>
      </div>
      <div class="method-detail-content">
        <div class="method-info-row">
          <div class="info-block">
            <span>适用源系统</span>
            <strong>{{ selectedMethod.sourceSystem }}</strong>
          </div>
          <div class="info-block">
            <span>对应业务页面</span>
            <strong>{{ selectedMethod.businessPage }}</strong>
          </div>
        </div>
        <div class="method-core">
          <strong>核心逻辑</strong>
          <p>{{ selectedMethod.coreLogic }}</p>
        </div>
        <div class="method-fields">
          <strong>录入字段要求</strong>
          <div class="field-list">
            <span v-for="f in selectedMethod.fields" :key="f.tag" class="field-chip" :class="{ required: f.required }">
              {{ f.name }}
            </span>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 系统接口全景图 ==================== -->
    <article class="card panel system-map-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">系统改造</p>
          <h3>外围系统接口对接全景</h3>
        </div>
        <span class="mini">管理会计平台为新建核心，其余系统为改造或对接</span>
      </div>

      <div class="sys-map">
        <div class="sys-center">
          <div class="sys-center-core">
            <strong>管理会计平台</strong>
            <small>新建 · 规则引擎 · 归集台账 · 分摊试算 · 多维分析</small>
          </div>
          <div class="sys-center-badge">本项目核心产出</div>
        </div>

        <!-- 左侧：源业务系统 -->
        <div class="sys-tier sys-tier-source">
          <div class="sys-tier-label">源业务系统</div>
          <div class="sys-card" style="--sc: #49dcb1">
            <div class="sys-card-head">
              <strong>费用报销系统</strong>
              <span class="sys-tag tag-transform">改造</span>
            </div>
            <p>新增客群/产品/活动/渠道/条线字段；直归判定标记</p>
            <div class="sys-fields">接口字段：customerCode, productCode, eventCode, channelCode, isDirect</div>
          </div>
          <div class="sys-card" style="--sc: #55c8ff">
            <div class="sys-card-head">
              <strong>CRM系统</strong>
              <span class="sys-tag tag-new">新增</span>
            </div>
            <p>新增客户经理工时登记；服务关系配置；管户AUM占比计算</p>
            <div class="sys-fields">接口字段：serverStaff, workHours, workHourRatio, serviceScope</div>
          </div>
          <div class="sys-card" style="--sc: #f7c46a">
            <div class="sys-card-head">
              <strong>营销管理平台</strong>
              <span class="sys-tag tag-transform">改造</span>
            </div>
            <p>活动编号管理；推广批次管理；转化线索自动归集</p>
            <div class="sys-fields">接口字段：eventCode, batchId, conversionCount, conversionAmount</div>
          </div>
          <div class="sys-card" style="--sc: #e879f9">
            <div class="sys-card-head">
              <strong>人力资源系统</strong>
              <span class="sys-tag tag-transform">改造</span>
            </div>
            <p>岗位-条线映射；工时采集接口；人员-客户/产品关系</p>
            <div class="sys-fields">接口字段：positionCode, lineCode, staffId, relationType</div>
          </div>
        </div>

        <!-- 右侧：数据供给系统 -->
        <div class="sys-tier sys-tier-supply">
          <div class="sys-tier-label">数据供给系统</div>
          <div class="sys-card" style="--sc: #55c8ff">
            <div class="sys-card-head">
              <strong>核心业务系统</strong>
              <span class="sys-tag tag-link">对接</span>
            </div>
            <p>交易量/客户数/资产数据；产品-机构归属映射；系统调用量日志</p>
            <div class="sys-fields">接口字段：transCount, transAmount, clientCount, productOrgMap</div>
          </div>
          <div class="sys-card" style="--sc: #f7c46a">
            <div class="sys-card-head">
              <strong>信贷管理系统</strong>
              <span class="sys-tag tag-new">新增</span>
            </div>
            <p>贷款余额/发放额/户数；信贷产品与条线映射；不良率关联成本</p>
            <div class="sys-fields">接口字段：loanBalance, loanAmount, creditProductCode, lineCode</div>
          </div>
          <div class="sys-card" style="--sc: #49dcb1">
            <div class="sys-card-head">
              <strong>财务系统</strong>
              <span class="sys-tag tag-transform">改造</span>
            </div>
            <p>公共费用原始台账；成本科目标准化；分摊规则审批状态回写</p>
            <div class="sys-fields">接口字段：costAccount, publicCostAmount, poolLevel, approvalStatus</div>
          </div>
          <div class="sys-card" style="--sc: #e879f9">
            <div class="sys-card-head">
              <strong>IT监控平台</strong>
              <span class="sys-tag tag-link">对接</span>
            </div>
            <p>设备开机时长；系统调用量日志；线上渠道行为日志</p>
            <div class="sys-fields">接口字段：deviceUpTime, apiCallCount, channelAccessLog</div>
          </div>
          <div class="sys-card" style="--sc: #55c8ff">
            <div class="sys-card-head">
              <strong>资产管理系统</strong>
              <span class="sys-tag tag-transform">改造</span>
            </div>
            <p>网点面积/设备台账；场地占用分摊因子；资产折旧成本归集</p>
            <div class="sys-fields">接口字段：branchArea, deviceCount, assetDepreciation</div>
          </div>
          <div class="sys-card" style="--sc: #f7c46a">
            <div class="sys-card-head">
              <strong>产品管理系统</strong>
              <span class="sys-tag tag-transform">改造</span>
            </div>
            <p>产品目录标准化；产品-条线映射；产品生命周期成本归集</p>
            <div class="sys-fields">接口字段：productCode, productLine, productLifecycle</div>
          </div>
        </div>

        <!-- 下侧：消费分析系统 -->
        <div class="sys-tier sys-tier-consume">
          <div class="sys-tier-label">消费分析系统（数据输出）</div>
          <div class="sys-card sys-card-small" style="--sc: #55c8ff">
            <div class="sys-card-head">
              <strong>经营分析平台</strong>
              <span class="sys-tag tag-link">对接</span>
            </div>
            <p>多维盈利分析；客群/产品/渠道ROI看板</p>
          </div>
          <div class="sys-card sys-card-small" style="--sc: #49dcb1">
            <div class="sys-card-head">
              <strong>数据仓库/BI</strong>
              <span class="sys-tag tag-link">对接</span>
            </div>
            <p>归集结果落标；成本主题宽表；定期ETL</p>
          </div>
          <div class="sys-card sys-card-small" style="--sc: #f7c46a">
            <div class="sys-card-head">
              <strong>监管报送系统</strong>
              <span class="sys-tag tag-link">对接</span>
            </div>
            <p>管理会计数据监管报告；成本分摊结果披露</p>
          </div>
        </div>

        <div class="sys-conn-left">
          <div class="conn-line"></div>
          <div class="conn-arrow">数据拉取</div>
        </div>
        <div class="sys-conn-right">
          <div class="conn-line"></div>
          <div class="conn-arrow">数据推送</div>
        </div>
      </div>
    </article>

  </section>
</template>

<style scoped>
.dashboard-layout {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

/* Hero */
.hero-panel { grid-column: 1 / -1; }
.method-hero { display: flex; flex-direction: column; gap: 20px; }
.method-title-area { }
.method-title-area h2 {
  margin: 8px 0 14px;
  font-size: 26px;
  background: linear-gradient(135deg, #49dcb1, #55c8ff, #f7c46a, #e879f9);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
.method-intro {
  color: var(--muted);
  line-height: 1.8;
  max-width: 900px;
  margin: 0;
}
.method-intro strong { color: var(--cyan); }

.process-chain {
  display: flex;
  align-items: center;
  gap: 0;
  padding: 16px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  overflow-x: auto;
}

.process-step {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  flex: 1;
  min-width: 100px;
  position: relative;
}
.process-step::after {
  content: '→';
  position: absolute;
  right: -12px;
  top: 10px;
  color: var(--muted);
  font-size: 16px;
}
.process-step:last-child::after { display: none; }
.process-dot {
  width: 12px; height: 12px; border-radius: 50%;
}
.process-step span { font-size: 12px; color: var(--muted); text-align: center; white-space: nowrap; }

/* 三层递进 */
.tiers-panel { grid-column: 1 / -1; }
.tiers-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 14px;
}

.tier-card {
  padding: 18px;
  border-radius: 18px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  transition: border-color .18s ease;
}
.tier-card:hover { border-color: var(--line-strong); }

.tier-header { display: flex; gap: 12px; align-items: flex-start; margin-bottom: 10px; }
.tier-icon { font-size: 24px; }
.tier-header strong { font-size: 14px; }
.tier-value {
  font-size: 11px;
  color: var(--cyan);
  margin: 4px 0 0;
}
.tier-desc {
  color: var(--muted);
  font-size: 12px;
  line-height: 1.7;
  margin: 0 0 12px;
}
.tier-targets { display: flex; flex-wrap: wrap; gap: 6px; }
.tier-tag {
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(85,200,255,.1);
  border: 1px solid var(--line);
  color: var(--text);
  font-size: 11px;
}

/* 四分法概览 */
.four-cat-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.cat-card {
  padding: 16px;
  border-radius: 16px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  cursor: pointer;
  transition: all .18s ease;
}
.cat-card:hover { border-color: var(--cat-color); }
.cat-card.active {
  border-color: var(--cat-color);
  background: color-mix(in srgb, var(--cat-color) 8%, transparent);
}

.cat-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px; }
.cat-head strong { font-size: 12px; color: var(--cat-color); }
.cat-tag { font-size: 11px; padding: 3px 8px; border-radius: 999px; background: rgba(255,255,255,.06); color: var(--muted); }
.cat-card h4 { font-size: 14px; margin: 0 0 8px; }
.cat-desc { color: var(--muted); font-size: 12px; line-height: 1.6; margin: 0 0 10px; }
.cat-system { border-top: 1px solid var(--line); padding-top: 8px; }
.cat-system small { color: var(--muted); font-size: 11px; }

/* 典型场景 */
.scenes-summary-panel { grid-column: 1 / -1; }
.scenes-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 12px;
}

.scene-card {
  padding: 16px;
  border-radius: 16px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
  cursor: pointer;
  transition: all .18s ease;
}
.scene-card:hover { border-color: var(--sc-color); }
.scene-card.active {
  border-color: var(--sc-color);
  background: color-mix(in srgb, var(--sc-color) 8%, transparent);
}

.scene-card-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.scene-card-head strong { font-size: 13px; color: var(--sc-color); }
.scene-method-tag {
  font-size: 10px;
  padding: 3px 8px;
  border-radius: 999px;
  background: color-mix(in srgb, var(--sc-color) 15%, transparent);
  color: var(--sc-color);
}
.scene-card p { color: var(--muted); font-size: 11px; line-height: 1.6; margin: 0 0 10px; }
.scene-amount { display: flex; justify-content: space-between; align-items: center; border-top: 1px solid var(--line); padding-top: 8px; }
.scene-amount span { font-size: 11px; color: var(--muted); }
.scene-amount strong { font-size: 14px; color: var(--sc-color); }

/* 对应关系表 */
.relation-panel { grid-column: 1 / -1; }

/* 方法详情 */
.method-detail-panel { border-color: var(--cat-color); }
.method-detail-content { display: flex; flex-direction: column; gap: 14px; }
.method-info-row { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 12px; }
.info-block { padding: 12px; border-radius: 12px; background: rgba(255,255,255,.02); border: 1px solid var(--line); }
.info-block span { display: block; color: var(--muted); font-size: 11px; margin-bottom: 4px; }
.info-block strong { font-size: 13px; color: var(--cat-color); }
.method-core { padding: 12px; border-radius: 12px; background: rgba(255,255,255,.02); border: 1px solid var(--line); }
.method-core strong { font-size: 12px; color: var(--cat-color); display: block; margin-bottom: 6px; }
.method-core p { color: var(--muted); font-size: 12px; line-height: 1.7; margin: 0; }
.method-fields strong { font-size: 12px; display: block; margin-bottom: 8px; }
.field-list { display: flex; flex-wrap: wrap; gap: 6px; }
.field-chip {
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(255,255,255,.05);
  border: 1px solid var(--line);
  font-size: 11px;
}
.field-chip.required { border-color: var(--cat-color); color: var(--cat-color); }

/* 系统接口全景图 */
.system-map-panel { grid-column: 1 / -1; }

.sys-map {
  position: relative;
  display: grid;
  grid-template-columns: 1fr 140px 1fr;
  gap: 16px;
  align-items: start;
}

.sys-center {
  grid-column: 2;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 6px;
  padding: 14px 10px;
  border-radius: 16px;
  background: linear-gradient(135deg, rgba(73,220,177,.12), rgba(85,200,255,.12));
  border: 2px solid rgba(85,200,255,.3);
  text-align: center;
}
.sys-center-core strong { display: block; font-size: 13px; color: var(--cyan); }
.sys-center-core small { display: block; font-size: 10px; color: var(--muted); margin-top: 4px; line-height: 1.4; }
.sys-center-badge {
  font-size: 10px;
  padding: 3px 10px;
  border-radius: 999px;
  background: rgba(73,220,177,.2);
  color: var(--teal);
}

.sys-tier { display: flex; flex-direction: column; gap: 10px; }
.sys-tier-label {
  font-size: 10px;
  color: var(--muted);
  text-transform: uppercase;
  letter-spacing: .08em;
  margin-bottom: 2px;
}

.sys-tier-consume {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 10px;
  padding-top: 8px;
  border-top: 1px dashed rgba(85,200,255,.15);
}

.sys-card {
  padding: 10px 12px;
  border-radius: 12px;
  background: rgba(255,255,255,.02);
  border: 1px solid rgba(255,255,255,.06);
  border-left: 3px solid var(--sc);
}
.sys-card-small { padding: 10px; }

.sys-card-head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 5px;
  gap: 6px;
}
.sys-card-head strong { font-size: 12px; color: var(--sc); flex: 1; }
.sys-card p { font-size: 11px; color: var(--muted); line-height: 1.5; margin: 0 0 5px; }

.sys-tag { font-size: 9px; padding: 2px 6px; border-radius: 4px; flex-shrink: 0; }
.sys-tag.tag-transform { background: rgba(85,200,255,.15); color: var(--cyan); }
.sys-tag.tag-new { background: rgba(247,196,106,.15); color: var(--gold); }
.sys-tag.tag-link { background: rgba(73,220,177,.12); color: var(--teal); }

.sys-fields {
  font-size: 10px;
  color: var(--muted);
  font-family: monospace;
  background: rgba(255,255,255,.03);
  padding: 4px 6px;
  border-radius: 6px;
  word-break: break-all;
}

.sys-conn-left,
.sys-conn-right {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
.sys-conn-left { left: -40px; }
.sys-conn-right { right: -40px; }
.conn-line { width: 30px; height: 2px; background: linear-gradient(90deg, transparent, rgba(85,200,255,.4)); }
.sys-conn-right .conn-line { background: linear-gradient(90deg, rgba(85,200,255,.4), transparent); }
.conn-arrow { font-size: 10px; color: var(--muted); }

@media (max-width: 1100px) {
  .sys-map { grid-template-columns: 1fr; }
  .sys-center { grid-column: 1; }
  .sys-tier-source { grid-column: 1; }
  .sys-tier-supply { grid-column: 1; }
  .sys-tier-consume { grid-column: 1; }
  .sys-conn-left, .sys-conn-right { display: none; }
}
</style>
