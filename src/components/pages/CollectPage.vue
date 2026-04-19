<script setup>
defineProps({
  methods: { type: Array, required: true },
  activeMethod: { type: String, required: true },
  selectedMethod: { type: Object, required: true },
  methodForm: { type: Object, required: true },
  directPreview: { type: Object, required: true },
  servicePreview: { type: Object, required: true },
  driverPreview: { type: Object, required: true },
  poolPreview: { type: Object, required: true },
  sourceEnhancements: { type: Array, required: true },
  money: { type: Function, required: true }
})

const emit = defineEmits(['change-method'])
</script>

<template>
  <section class="page-grid collect-layout">
    <article class="card panel method-overview-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">场景选择</p>
          <h3>归集方法业务改造视图</h3>
        </div>
        <span class="mini">按方法进入不同业务页面</span>
      </div>

      <div class="method-tabs">
        <button
          v-for="item in methods"
          :key="item.id"
          class="method-tab"
          :class="{ active: item.id === activeMethod }"
          @click="emit('change-method', item.id)"
        >
          <strong>{{ item.label }}</strong>
          <small>{{ item.scenario }}</small>
        </button>
      </div>

      <div class="scene-banner">
        <div>
          <strong>{{ selectedMethod.businessPage }}</strong>
          <p>{{ selectedMethod.scenarioDescription }}</p>
        </div>
        <div class="scene-banner-meta">
          <span class="chip">{{ selectedMethod.sourceSystem }}</span>
          <span class="chip">{{ selectedMethod.ruleCode }}</span>
        </div>
      </div>
    </article>

    <article class="card panel method-form-panel">
      <div class="title-row">
        <div>
          <p class="eyebrow">{{ selectedMethod.businessPage }}</p>
          <h3>{{ selectedMethod.formTitle }}</h3>
        </div>
        <span class="mini">模拟现有业务系统改造后录入</span>
      </div>

      <div class="form-grid form-grid-3">
        <label><span>业务单据号</span><input v-model="methodForm.documentNo" /></label>
        <label><span>成本科目</span><input v-model="methodForm.costSubject" /></label>
        <label><span>发生金额</span><input v-model.number="methodForm.amount" type="number" /></label>
        <label><span>归属机构</span><input v-model="methodForm.org" /></label>
        <label><span>归属条线</span><input v-model="methodForm.line" /></label>
        <label><span>归属渠道</span><input v-model="methodForm.channel" /></label>
        <label><span>产品编码</span><input v-model="methodForm.productCode" /></label>
        <label><span>客户/客群</span><input v-model="methodForm.customerSegment" /></label>
        <label><span>活动/项目编号</span><input v-model="methodForm.eventCode" /></label>
        <label><span>责任经理/岗位</span><input v-model="methodForm.owner" /></label>
        <label><span>{{ selectedMethod.primaryFieldLabel }}</span><input v-model="methodForm.primaryField" /></label>
        <label><span>{{ selectedMethod.secondaryFieldLabel }}</span><input v-model="methodForm.secondaryField" /></label>
      </div>

      <label class="notes">
        <span>业务说明 / 受益对象说明</span>
        <textarea v-model="methodForm.notes" rows="4" />
      </label>
    </article>

    <article class="card panel flow-panel">
      <div class="title-row">
        <h3>归集处理逻辑</h3>
        <span class="mini">不是调比例，而是进入对应业务归集页面</span>
      </div>

      <div class="flow-steps">
        <div class="flow-step">
          <span class="step-no">01</span>
          <div>
            <strong>源单识别</strong>
            <p>{{ selectedMethod.identifyLogic }}</p>
          </div>
        </div>
        <div class="flow-step">
          <span class="step-no">02</span>
          <div>
            <strong>字段校验</strong>
            <p>{{ selectedMethod.validationLogic }}</p>
          </div>
        </div>
        <div class="flow-step">
          <span class="step-no">03</span>
          <div>
            <strong>归集落点</strong>
            <p>{{ selectedMethod.allocationLogic }}</p>
          </div>
        </div>
      </div>
    </article>

    <article class="card panel preview-panel">
      <div class="title-row">
        <h3>成本归集结果预览</h3>
        <span class="mini">按方法返回不同对象</span>
      </div>

      <div v-if="activeMethod === 'direct'" class="preview-card-grid">
        <div class="preview-card accent-direct">
          <span>直接归集对象</span>
          <strong>{{ directPreview.target }}</strong>
          <p>{{ directPreview.logic }}</p>
          <b>{{ money(directPreview.amount) }}</b>
        </div>
        <div class="preview-card">
          <span>写入台账</span>
          <strong>{{ directPreview.ledger }}</strong>
          <p>{{ directPreview.trace }}</p>
        </div>
      </div>

      <div v-else-if="activeMethod === 'service'" class="preview-card-grid">
        <div class="preview-card accent-service">
          <span>服务关系主对象</span>
          <strong>{{ servicePreview.primary }}</strong>
          <p>{{ servicePreview.logic }}</p>
          <b>{{ money(servicePreview.primaryAmount) }}</b>
        </div>
        <div class="preview-card">
          <span>辅助对象</span>
          <strong>{{ servicePreview.secondary }}</strong>
          <p>{{ servicePreview.trace }}</p>
          <b>{{ money(servicePreview.secondaryAmount) }}</b>
        </div>
      </div>

      <div v-else-if="activeMethod === 'driver'" class="preview-card-grid">
        <div v-for="item in driverPreview.targets" :key="item.name" class="preview-card accent-driver">
          <span>{{ item.driver }}</span>
          <strong>{{ item.name }}</strong>
          <p>{{ item.ratio }}</p>
          <b>{{ money(item.amount) }}</b>
        </div>
      </div>

      <div v-else class="preview-card-grid">
        <div v-for="item in poolPreview.targets" :key="item.name" class="preview-card accent-pool">
          <span>{{ item.rule }}</span>
          <strong>{{ item.name }}</strong>
          <p>{{ item.base }}</p>
          <b>{{ money(item.amount) }}</b>
        </div>
      </div>
    </article>

    <article class="card panel enhance-panel">
      <div class="title-row">
        <h3>现有系统需要改造的字段/流程</h3>
        <span class="mini">围绕当前方法补齐采集能力</span>
      </div>
      <div class="check-list">
        <div v-for="item in sourceEnhancements" :key="item.title">
          <strong>{{ item.title }}</strong>
          <p>{{ item.desc }}</p>
        </div>
      </div>
    </article>
  </section>
</template>
