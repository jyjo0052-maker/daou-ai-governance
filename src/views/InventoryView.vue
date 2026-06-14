<script setup>
import { ref, computed } from 'vue'
import { aiServices } from '../data/services.js'

const props = defineProps({ onOpenDetail: Function })

const selectedProduct = ref('전체')
const selectedRisk = ref('전체')
const selectedStatus = ref('전체')
const searchQuery = ref('')

const QUALITY_THRESHOLD = 85

const products = computed(() => ['전체', ...new Set(aiServices.map(s => s.product))])
const riskLevels = ['전체', '고영향 AI', '생성형 AI', '일반 AI']
const statusOptions = ['전체', '정상', '주의', '점검필요']

const filteredServices = computed(() =>
  aiServices.filter(s => {
    const matchProduct = selectedProduct.value === '전체' || s.product === selectedProduct.value
    const matchRisk = selectedRisk.value === '전체' || s.riskLevel === selectedRisk.value
    const matchStatus = selectedStatus.value === '전체' || s.status === selectedStatus.value
    const matchSearch = searchQuery.value === '' ||
      s.feature.includes(searchQuery.value) || s.model.includes(searchQuery.value) || s.product.includes(searchQuery.value)
    return matchProduct && matchRisk && matchStatus && matchSearch
  })
)

function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }
function getStatusClass(s) { return { '정상': 'status-ok', '주의': 'status-warn', '점검필요': 'status-danger' }[s] || '' }
function getScoreClass(score) { return score >= 90 ? 'score-high' : score >= 85 ? 'score-mid' : 'score-low' }
function getComplianceStatus(s) {
  const vals = Object.values(s.compliance)
  const passed = vals.filter(Boolean).length
  if (passed === vals.length) return { label: '준수', cls: 'comp-ok' }
  if (passed >= vals.length - 1) return { label: '일부 미준수', cls: 'comp-warn' }
  return { label: '미준수', cls: 'comp-fail' }
}
</script>

<template>
  <div>
    <div class="view-header">
      <div>
        <h2 class="view-title">AI 서비스 인벤토리</h2>
        <p class="view-desc">다우기술 전체 AI 서비스 목록입니다. 필터와 검색으로 원하는 서비스를 찾고 상세 정보를 확인하세요.</p>
      </div>
      <button class="btn-primary-sm">
        <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
        신규 AI 등록
      </button>
    </div>

    <div class="filters">
      <div class="filter-group">
        <label class="filter-label">제품</label>
        <select v-model="selectedProduct" class="filter-select">
          <option v-for="p in products" :key="p">{{ p }}</option>
        </select>
      </div>
      <div class="filter-group">
        <label class="filter-label">등급</label>
        <select v-model="selectedRisk" class="filter-select">
          <option v-for="r in riskLevels" :key="r">{{ r }}</option>
        </select>
      </div>
      <div class="filter-group">
        <label class="filter-label">상태</label>
        <select v-model="selectedStatus" class="filter-select">
          <option v-for="s in statusOptions" :key="s">{{ s }}</option>
        </select>
      </div>
      <div class="filter-group filter-search">
        <label class="filter-label">검색</label>
        <input v-model="searchQuery" type="text" class="filter-input" placeholder="서비스명, 모델명..." />
      </div>
    </div>

    <section class="table-section">
      <div class="table-wrap">
        <table class="data-table">
          <thead>
            <tr>
              <th>제품</th>
              <th>AI 기능</th>
              <th>모델</th>
              <th>등급</th>
              <th>품질점수</th>
              <th>규제 준수</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="s in filteredServices" :key="s.id" @click="onOpenDetail(s)" class="table-row">
              <td><span class="product-tag" :class="'product-' + s.product">{{ s.product }}</span></td>
              <td>
                <div class="feature-cell">
                  <span class="feature-name">{{ s.feature }}</span>
                  <span class="feature-desc">{{ s.description }}</span>
                </div>
              </td>
              <td>
                <div class="model-cell">
                  <span class="model-name">{{ s.model }}</span>
                  <span class="model-version">{{ s.modelVersion }}</span>
                </div>
              </td>
              <td><span class="risk-badge" :class="getRiskClass(s.riskLevel)">{{ s.riskLevel }}</span></td>
              <td>
                <div class="score-cell">
                  <span class="score-value" :class="getScoreClass(s.qualityScore)">{{ s.qualityScore }}</span>
                  <div class="score-bar">
                    <div class="score-bar-fill" :class="getScoreClass(s.qualityScore)" :style="{ width: s.qualityScore + '%' }"></div>
                  </div>
                  <span v-if="s.qualityScore < QUALITY_THRESHOLD" class="threshold-warn">▲</span>
                </div>
              </td>
              <td>
                <span class="comp-badge" :class="getComplianceStatus(s).cls">
                  {{ getComplianceStatus(s).label }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table-footer"><span class="result-count">총 {{ filteredServices.length }}개 서비스</span></div>
    </section>
  </div>
</template>
