<script setup>
import { ref, computed } from 'vue'
import { aiServices, complianceItems } from '../data/services.js'

const selectedProduct = ref('전체')
const products = computed(() => ['전체', ...new Set(aiServices.map(s => s.product))])

const filteredServices = computed(() =>
  selectedProduct.value === '전체' ? aiServices : aiServices.filter(s => s.product === selectedProduct.value)
)

function isApplicable(s, key) {
  if (key === 'generativeAILabel') return s.riskLevel === '생성형 AI' || s.riskLevel === '고영향 AI'
  return true
}

const totalViolations = computed(() => {
  let count = 0
  filteredServices.value.forEach(s => {
    complianceItems.forEach(item => {
      if (isApplicable(s, item.key) && !s.compliance[item.key]) count++
    })
  })
  return count
})

const fullyCompliant = computed(() =>
  filteredServices.value.filter(s =>
    complianceItems.every(item => !isApplicable(s, item.key) || s.compliance[item.key])
  ).length
)

function getItemStatus(service, key) { return service.compliance[key] }
function getServiceComplianceScore(s) {
  const applicable = complianceItems.filter(item => isApplicable(s, item.key))
  return Math.round((applicable.filter(item => s.compliance[item.key]).length / applicable.length) * 100)
}
function getScoreClass(score) { return score === 100 ? 'score-high' : score >= 67 ? 'score-mid' : 'score-low' }

const downloadReport = () => { alert('감사 대응 리포트 다운로드 (PDF)\n실제 배포 시 자동 생성됩니다.') }
</script>

<template>
  <div>
    <!-- Header -->
    <div class="view-header">
      <div>
        <h2 class="view-title">규제 준수 현황</h2>
        <p class="view-desc">AI 기본법·개인정보보호법 기반 의무 이행 현황을 확인하고 외부 감사 대응 자료를 생성합니다.</p>
      </div>
      <button class="btn-primary-sm" @click="downloadReport">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
        감사 대응 리포트 다운로드
      </button>
    </div>

    <!-- Summary Cards -->
    <div class="kpi-section" style="margin-bottom:24px">
      <div class="kpi-card" :class="{ 'kpi-alert': totalViolations > 0 }">
        <div class="kpi-header">
          <span class="kpi-label">규제 위반 항목</span>
          <div class="kpi-icon kpi-icon-red">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="15" y1="9" x2="9" y2="15"/><line x1="9" y1="9" x2="15" y2="15"/></svg>
          </div>
        </div>
        <div class="kpi-value">{{ totalViolations }}<span class="kpi-unit">건</span></div>
        <div class="kpi-sub">즉시 조치 필요</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-header">
          <span class="kpi-label">완전 준수 서비스</span>
          <div class="kpi-icon kpi-icon-green">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
          </div>
        </div>
        <div class="kpi-value">{{ fullyCompliant }}<span class="kpi-unit"> / {{ filteredServices.length }}</span></div>
        <div class="kpi-sub">전체 대비 {{ Math.round(fullyCompliant/filteredServices.length*100) }}%</div>
      </div>
      <div class="kpi-card">
        <div class="kpi-header">
          <span class="kpi-label">적용 규제</span>
          <div class="kpi-icon kpi-icon-blue">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          </div>
        </div>
        <div class="kpi-value" style="font-size:18px;line-height:1.4">AI 기본법<br><span style="font-size:14px;color:var(--gray-500)">+ 개인정보보호법</span></div>
        <div class="kpi-sub">2026.01.22 시행</div>
      </div>
    </div>

    <!-- Filter -->
    <div class="filters" style="margin-bottom:20px">
      <div class="filter-group">
        <label class="filter-label">제품 필터</label>
        <select v-model="selectedProduct" class="filter-select">
          <option v-for="p in products" :key="p">{{ p }}</option>
        </select>
      </div>
    </div>

    <!-- Compliance Matrix -->
    <section class="table-section">
      <h3 class="section-title" style="margin-bottom:16px">의무 항목별 준수 현황</h3>

      <!-- Legend -->
      <div class="comp-legend">
        <span class="legend-item"><span class="legend-dot dot-ok"></span> 이행 완료</span>
        <span class="legend-item"><span class="legend-dot dot-fail"></span> 미이행 (즉시 조치 필요)</span>
      </div>

      <div class="table-wrap">
        <table class="data-table compliance-table">
          <thead>
            <tr>
              <th style="min-width:140px">서비스</th>
              <th v-for="item in complianceItems" :key="item.key" class="comp-th">
                <div class="comp-th-inner">
                  <span>{{ item.label }}</span>
                  <span class="comp-law">{{ item.law }}</span>
                </div>
              </th>
              <th>준수율</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="s in filteredServices" :key="s.id">
              <td>
                <div class="feature-cell">
                  <span class="product-tag" :class="'product-' + s.product" style="margin-bottom:4px;display:inline-block">{{ s.product }}</span>
                  <span class="feature-name" style="display:block">{{ s.feature }}</span>
                </div>
              </td>
              <td v-for="item in complianceItems" :key="item.key" class="comp-cell">
                <div v-if="!isApplicable(s, item.key)" class="comp-check comp-na" title="해당 없음 (생성형 AI 아님)">
                  —
                </div>
                <div v-else-if="getItemStatus(s, item.key)" class="comp-check comp-pass">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
                </div>
                <div v-else class="comp-check comp-fail-icon" :title="`${item.label} 미이행 — ${item.desc}`">
                  <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
                </div>
              </td>
              <td>
                <div class="score-cell">
                  <span class="score-value" :class="getScoreClass(getServiceComplianceScore(s))">{{ getServiceComplianceScore(s) }}%</span>
                  <div class="score-bar" style="width:50px">
                    <div class="score-bar-fill" :class="getScoreClass(getServiceComplianceScore(s))" :style="{ width: getServiceComplianceScore(s) + '%' }"></div>
                  </div>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- Regulation Reference -->
    <section class="table-section">
      <h3 class="section-title" style="margin-bottom:16px">규제 의무 항목 상세</h3>
      <div class="reg-list">
        <div v-for="item in complianceItems" :key="item.key" class="reg-item">
          <div class="reg-left">
            <span class="reg-law">{{ item.law }}</span>
            <span class="reg-label">{{ item.label }}</span>
          </div>
          <p class="reg-desc">{{ item.desc }}</p>
          <div class="reg-status">
            <span class="reg-pass">{{ filteredServices.filter(s => s.compliance[item.key]).length }}개 이행</span>
            <span class="reg-fail" v-if="filteredServices.filter(s => !s.compliance[item.key]).length > 0">
              {{ filteredServices.filter(s => !s.compliance[item.key]).length }}개 미이행
            </span>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
