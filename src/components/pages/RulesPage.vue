<script setup>
defineProps({
  ruleGroups: { type: Array, required: true },
  fourCategories: { type: Array, required: true },
  activeMethod: { type: String, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-method'])

const ruleDesignPrinciples = [
  { title: '直接归集优先级前置', desc: '能够通过客户、产品、活动、账户直接识别归属的成本，不再进入统一分摊池，优先使用直接归集规则。', tag: '优先级' },
  { title: '场景化规则簇组织', desc: '围绕客户经营、产品推动、渠道运营、公共管理等主题组织规则簇，避免只按会计科目做静态分摊。', tag: '场景化' },
  { title: '驱动因子可追溯', desc: '所有分摊规则绑定来源系统、字段口径、采集频率与责任部门，便于后续审计和优化。', tag: '可审计' },
  { title: '规则版本化管理', desc: '规则变更需记录版本号、变更原因、审批状态，确保历史规则可追溯，不同时期规则可对比。', tag: '版本化' },
  { title: '规则固化到系统参数', desc: '方法论不是停留在文档，而是转换成可配置的规则版本、试算开关与审批流程。', tag: '可落地' },
  { title: '试算验证优先', desc: '规则配置后需执行试算，对比历史数据，验证合理性后方可生效，避免上线即出错。', tag: '可验证' }
]
</script>

<template>
  <section class="page-grid rules-layout">

    <!-- ==================== 规则设计原则 ==================== -->
    <article class="card panel principles-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">规则引擎设计</p>
          <h3>场景化规则库设计原则</h3>
        </div>
        <span class="mini">不是做静态分摊表，而是可配置、可验证、可追溯的规则簇</span>
      </div>
      <div class="principles-grid">
        <div v-for="p in ruleDesignPrinciples" :key="p.title" class="principle-card">
          <div class="principle-head">
            <strong>{{ p.title }}</strong>
            <span class="chip">{{ p.tag }}</span>
          </div>
          <p>{{ p.desc }}</p>
        </div>
      </div>
    </article>

    <!-- ==================== 四分法规则分组选择 ==================== -->
    <article class="card panel method-selector-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">规则分类</p>
          <h3>按四分法选择规则簇</h3>
        </div>
      </div>
      <div class="method-selector">
        <button
          v-for="cat in fourCategories"
          :key="cat.id"
          class="method-selector-btn"
          :class="{ active: activeMethod === cat.id }"
          :style="{ '--cat-color': cat.color }"
          @click="emit('change-method', cat.id)"
        >
          <strong>{{ cat.label }}</strong>
          <small>{{ cat.name }}</small>
          <span class="rule-count">{{ cat.id === 'direct' ? 4 : cat.id === 'service' ? 4 : cat.id === 'driver' ? 4 : 4 }}条规则</span>
        </button>
      </div>
    </article>

    <!-- ==================== 规则列表 ==================== -->
    <article class="card panel rules-list-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">规则明细</p>
          <h3>规则库明细</h3>
        </div>
        <span class="mini">点击分类查看规则详情</span>
      </div>

      <div class="rules-sections">
        <div
          v-for="group in ruleGroups"
          :key="group.title"
          class="rule-section"
          :class="{ highlighted: activeMethod === group.code.split('-')[1].toLowerCase() || activeMethod === group.code.split('-')[1].toLowerCase() }"
          :style="{ '--group-color': group.color }"
        >
          <div class="rule-section-head">
            <strong>{{ group.title }}</strong>
            <span class="code-badge">{{ group.code }}-XX</span>
          </div>
          <div class="rule-cards">
            <div v-for="rule in group.rules" :key="rule.code" class="rule-card">
              <div class="rule-card-head">
                <span class="rule-code">{{ rule.code }}</span>
                <strong>{{ rule.name }}</strong>
                <span class="rule-amount">{{ money(rule.cost) }}</span>
              </div>
              <p>{{ rule.desc }}</p>
            </div>
          </div>
        </div>
      </div>
    </article>

    <!-- ==================== 规则配置面板 ==================== -->
    <article class="card panel config-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">规则配置</p>
          <h3>规则参数配置</h3>
        </div>
        <span class="mini">当前选中：{{ fourCategories.find(c => c.id === activeMethod)?.name }}</span>
      </div>

      <div class="config-area">
        <div class="config-header">
          <div class="config-tag" :style="{ color: fourCategories.find(c => c.id === activeMethod)?.color }">
            {{ fourCategories.find(c => c.id === activeMethod)?.label }}
          </div>
          <span class="config-title">{{ fourCategories.find(c => c.id === activeMethod)?.name }}</span>
        </div>

        <div class="config-form">
          <div class="config-field">
            <label>规则编码</label>
            <input :value="fourCategories.find(c => c.id === activeMethod)?.id === 'direct' ? 'FR-D001' :
              fourCategories.find(c => c.id === activeMethod)?.id === 'service' ? 'FR-S001' :
              fourCategories.find(c => c.id === activeMethod)?.id === 'driver' ? 'FR-V001' : 'FR-P001'" readonly />
          </div>
          <div class="config-field">
            <label>规则名称</label>
            <input :value="fourCategories.find(c => c.id === activeMethod)?.id === 'direct' ? '客户直归法' :
              fourCategories.find(c => c.id === activeMethod)?.id === 'service' ? '客户贡献度归集法' :
              fourCategories.find(c => c.id === activeMethod)?.id === 'driver' ? '授信转化分摊法' : '总行管理费分配规则'" readonly />
          </div>
          <div class="config-field">
            <label>数据来源</label>
            <input :value="fourCategories.find(c => c.id === activeMethod)?.sourceSystem?.split('/')[0] || ''" readonly />
          </div>
          <div class="config-field">
            <label>输出台账</label>
            <input :value="activeMethod === 'direct' ? '归集台账-客户维/产品维/活动维' :
              activeMethod === 'service' ? '归集台账-服务关系维' :
              activeMethod === 'driver' ? '归集台账-驱动因子维' : '归集台账-公共池维'" readonly />
          </div>
          <div class="config-field">
            <label>生效状态</label>
            <select>
              <option>生效</option>
              <option>试算中</option>
              <option>已停用</option>
            </select>
          </div>
          <div class="config-field">
            <label>版本号</label>
            <input value="V2.1.0" readonly />
          </div>
        </div>

        <div class="config-params">
          <h4>配置参数</h4>
          <div class="param-list">
            <div class="param-item" v-for="f in (fourCategories.find(c => c.id === activeMethod)?.fields || []).slice(0, 4)" :key="f.tag">
              <span>{{ f.name }}</span>
              <div class="param-value">
                <input :placeholder="'输入 ' + f.name" />
                <small v-if="f.required" class="required-mark">必填</small>
              </div>
            </div>
          </div>
        </div>

        <div class="config-actions">
          <button class="btn-trial">执行试算</button>
          <button class="btn-save">保存规则</button>
          <button class="btn-history">查看历史版本</button>
        </div>
      </div>
    </article>

  </section>
</template>

<style scoped>
.rules-layout {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}

.principles-panel { grid-column: 1 / -1; }

.principles-grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
}

.principle-card {
  padding: 14px;
  border-radius: 14px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.principle-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 8px; }
.principle-head strong { font-size: 13px; }
.principle-card p { color: var(--muted); font-size: 12px; line-height: 1.6; margin: 0; }

/* 方法选择 */
.method-selector {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
}

.method-selector-btn {
  padding: 16px;
  border-radius: 14px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  color: var(--text);
  cursor: pointer;
  text-align: left;
  transition: all .18s ease;
}
.method-selector-btn:hover { border-color: var(--cat-color); }
.method-selector-btn.active {
  border-color: var(--cat-color);
  background: color-mix(in srgb, var(--cat-color) 10%, transparent);
}
.method-selector-btn strong { display: block; font-size: 13px; color: var(--cat-color); }
.method-selector-btn small { display: block; color: var(--muted); font-size: 11px; margin: 4px 0; }
.rule-count { font-size: 11px; color: var(--muted); }

/* 规则列表 */
.rules-list-panel { grid-column: 1 / -1; }

.rules-sections { display: grid; gap: 14px; }

.rule-section {
  border-radius: 16px;
  border: 1px solid var(--line);
  background: rgba(255,255,255,.02);
  padding: 16px;
  transition: border-color .18s ease;
}
.rule-section.highlighted {
  border-color: var(--group-color);
  background: color-mix(in srgb, var(--group-color) 5%, transparent);
}

.rule-section-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}
.rule-section-head strong { font-size: 14px; }
.code-badge {
  padding: 4px 10px;
  border-radius: 999px;
  background: rgba(255,255,255,.06);
  color: var(--muted);
  font-size: 12px;
}

.rule-cards { display: grid; gap: 10px; }

.rule-card {
  padding: 12px 14px;
  border-radius: 12px;
  background: rgba(255,255,255,.02);
  border: 1px solid var(--line);
}
.rule-card-head { display: flex; align-items: center; gap: 10px; margin-bottom: 6px; }
.rule-code {
  padding: 3px 8px;
  border-radius: 6px;
  background: rgba(85,200,255,.12);
  color: var(--cyan);
  font-size: 11px;
  font-family: monospace;
}
.rule-card-head strong { font-size: 13px; flex: 1; }
.rule-amount { font-size: 12px; color: var(--muted); }
.rule-card p { color: var(--muted); font-size: 12px; line-height: 1.6; margin: 0; }

/* 规则配置 */
.config-area {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.config-header { display: flex; align-items: center; gap: 10px; }
.config-tag { font-size: 12px; font-weight: bold; }
.config-title { font-size: 14px; }

.config-form {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 10px;
}

.config-field { display: flex; flex-direction: column; gap: 6px; }
.config-field label { font-size: 11px; color: var(--muted); }
.config-field input, .config-field select {
  padding: 8px 10px;
  border-radius: 10px;
  border: 1px solid var(--line-strong);
  background: rgba(255,255,255,.04);
  color: var(--text);
  font-size: 12px;
}

.config-params h4 { font-size: 13px; margin-bottom: 10px; }
.param-list { display: grid; gap: 8px; }
.param-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 10px;
  padding: 8px 10px;
  border-radius: 10px;
  background: rgba(255,255,255,.02);
}
.param-item span { font-size: 12px; color: var(--muted); flex-shrink: 0; }
.param-value { display: flex; align-items: center; gap: 6px; flex: 1; justify-content: flex-end; }
.param-value input {
  width: 120px;
  padding: 6px 8px;
  border-radius: 8px;
  border: 1px solid var(--line-strong);
  background: rgba(255,255,255,.04);
  color: var(--text);
  font-size: 11px;
  text-align: right;
}
.required-mark { font-size: 10px; color: #ff6b6b; }

.config-actions { display: flex; gap: 10px; padding-top: 4px; }
.config-actions button {
  padding: 8px 16px;
  border-radius: 10px;
  border: 1px solid;
  cursor: pointer;
  font-size: 12px;
  transition: all .15s ease;
}
.btn-trial { background: rgba(85,200,255,.12); border-color: var(--line-strong); color: var(--cyan); }
.btn-trial:hover { background: rgba(85,200,255,.2); }
.btn-save { background: rgba(73,220,177,.12); border-color: rgba(73,220,177,.3); color: var(--teal); }
.btn-save:hover { background: rgba(73,220,177,.2); }
.btn-history { background: rgba(255,255,255,.04); border-color: var(--line); color: var(--muted); }
.btn-history:hover { background: rgba(255,255,255,.08); }

@media (max-width: 1100px) {
  .principles-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .method-selector { grid-template-columns: 1fr; }
  .config-form { grid-template-columns: 1fr; }
}
</style>
