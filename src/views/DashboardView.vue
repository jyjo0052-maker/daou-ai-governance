<script setup>
import { computed } from 'vue'
import { aiServices, complianceItems, recentChanges } from '../data/services.js'

const props = defineProps({ onOpenDetail: Function, onNavigate: Function })

const QUALITY_THRESHOLD = 85

// ── 전체 지표 ──
const totalServices = computed(() => aiServices.length)
const products = computed(() => [...new Set(aiServices.map(s => s.product))])

const overallComplianceRate = computed(() => {
  let pass = 0, total = 0
  aiServices.forEach(s => complianceItems.forEach(item => {
    if (item.key !== 'generativeAILabel' || s.riskLevel === '생성형 AI' || s.riskLevel === '고영향 AI') {
      total++
      if (s.compliance[item.key]) pass++
    }
  }))
  return total ? Math.round(pass / total * 100) : 100
})

const complianceViolationCount = computed(() =>
  aiServices.filter(s => Object.values(s.compliance).some(v => !v)).length
)
const alertServices = computed(() =>
  aiServices.filter(s => s.status === '주의' || s.status === '점검필요' || Object.values(s.compliance).some(v => !v))
)
const belowThreshold = computed(() => aiServices.filter(s => s.qualityScore < QUALITY_THRESHOLD).length)
function findService(c) { return aiServices.find(s => s.id === c.serviceId) }

// ── 등급별 현황 ──
const riskGroups = computed(() => {
  const levels = ['고영향 AI', '생성형 AI', '일반 AI']
  return levels.map(level => {
    const services = aiServices.filter(s => s.riskLevel === level)
    const compliant = services.filter(s =>
      complianceItems.every(item => {
        if (item.key === 'generativeAILabel' && level === '일반 AI') return true
        return s.compliance[item.key]
      })
    ).length
    return { level, count: services.length, compliant, rate: services.length ? Math.round(compliant / services.length * 100) : 100 }
  })
})

// ── 제품별 거버넌스 현황 ──
const productStats = computed(() =>
  products.value.map(p => {
    const services = aiServices.filter(s => s.product === p)
    const alertCount = services.filter(s => s.status === '주의' || s.status === '점검필요' || Object.values(s.compliance).some(v => !v)).length
    const pendingCRs = recentChanges.filter(c => c.product === p && c.status === '진행 중').length
    const compliant = services.filter(s => complianceItems.every(item => {
      if (item.key === 'generativeAILabel' && s.riskLevel === '일반 AI') return true
      return s.compliance[item.key]
    })).length
    const rate = Math.round(compliant / services.length * 100)
    return { product: p, count: services.length, rate, alertCount, pendingCRs }
  })
)

function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }
function getChangeStatusClass(s) { return { '승인': 'badge-success', '완료': 'badge-info', '진행 중': 'badge-warning' }[s] || 'badge-default' }
function getAlertReasons(s) {
  const r = []
  if (s.status === '주의' || s.status === '점검필요') r.push('품질 ' + s.status)
  const v = Object.values(s.compliance).filter(v => !v).length
  if (v > 0) r.push(`규제 미준수 ${v}건`)
  return r
}
function getCRStatusClass(s) { return { '진행 중': 'badge-warning', '검토 중': 'badge-default', '승인': 'badge-success' }[s] || 'badge-default' }
</script>

<template>
  <div>
    <!-- 헤더 -->
    <div class="view-header">
      <div>
        <h2 class="view-title">AI 거버넌스 현황</h2>
        <p class="view-desc">다우기술 전사 AI 서비스의 규제 준수·품질·변경 요청 현황을 통합 관리합니다.</p>
      </div>
      <span class="dashboard-date">기준일: 2026-06-14</span>
    </div>

    <!-- KPI 4개 -->
    <section class="kpi-section">
      <div class="kpi-card kpi-clickable" @click="onNavigate('inventory')">
        <div class="kpi-header">
          <span class="kpi-label">운영 중 AI 서비스</span>
          <div class="kpi-icon kpi-icon-blue">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8M12 17v4"/></svg>
          </div>
        </div>
        <div class="kpi-value">{{ totalServices }}<span class="kpi-unit">개</span></div>
        <div class="kpi-sub">{{ products.length }}개 제품에서 운영 중</div>
      </div>

      <div class="kpi-card kpi-clickable" :class="complianceViolationCount > 0 ? 'kpi-alert' : 'kpi-good'" @click="onNavigate('compliance')">
        <div class="kpi-header">
          <span class="kpi-label">규제 점검 필요</span>
          <div class="kpi-icon" :class="complianceViolationCount > 0 ? 'kpi-icon-red' : 'kpi-icon-green'">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          </div>
        </div>
        <template v-if="complianceViolationCount > 0">
          <div class="kpi-value">{{ complianceViolationCount }}<span class="kpi-unit">건</span></div>
          <div class="kpi-sub">규제 미준수 서비스</div>
        </template>
        <div v-else class="kpi-ok-state">
          <svg class="kpi-ok-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
          <span class="kpi-ok-text">모든 규제를 준수하고 있습니다</span>
        </div>
      </div>

      <div class="kpi-card kpi-clickable" :class="alertServices.length > 0 ? 'kpi-alert' : 'kpi-good'" @click="onNavigate('operations')">
        <div class="kpi-header">
          <span class="kpi-label">즉시 조치 필요</span>
          <div class="kpi-icon" :class="alertServices.length > 0 ? 'kpi-icon-orange' : 'kpi-icon-green'">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
          </div>
        </div>
        <template v-if="alertServices.length > 0">
          <div class="kpi-value">{{ alertServices.length }}<span class="kpi-unit">건</span></div>
          <div class="kpi-sub">품질 미달 {{ belowThreshold }}건 포함</div>
        </template>
        <div v-else class="kpi-ok-state">
          <svg class="kpi-ok-icon" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
          <span class="kpi-ok-text">모든 서비스 정상 운영 중</span>
        </div>
      </div>

      <div class="kpi-card">
        <div class="kpi-header">
          <span class="kpi-label">이번달 변경 이력</span>
          <div class="kpi-icon kpi-icon-blue">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
          </div>
        </div>
        <div class="kpi-value">{{ recentChanges.length }}<span class="kpi-unit">건</span></div>
        <div class="kpi-sub">전사 AI 서비스 변경 발생</div>
      </div>
    </section>

    <!-- 등급별 현황 + 제품별 거버넌스 -->
    <div class="dashboard-bottom-grid" style="margin-bottom:24px">

      <!-- 등급별 AI 서비스 현황 -->
      <section class="table-section">
        <h3 class="section-title" style="margin-bottom:16px">등급별 현황</h3>
        <div class="risk-group-list">
          <div v-for="g in riskGroups" :key="g.level" class="risk-group-item">
            <div class="risk-group-left">
              <span class="risk-badge" :class="getRiskClass(g.level)">{{ g.level }}</span>
              <span class="risk-group-count">{{ g.count }}개 서비스</span>
            </div>
            <div class="risk-group-right">
              <div class="risk-group-bar-wrap">
                <div class="risk-group-bar">
                  <div class="risk-group-bar-fill" :class="g.rate === 100 ? 'fill-green' : 'fill-red'" :style="{ width: g.rate + '%' }"></div>
                </div>
                <span class="risk-group-rate" :class="g.rate === 100 ? 'score-high' : 'score-low'">규제 준수 {{ g.rate }}%</span>
              </div>
            </div>
          </div>
        </div>
      </section>

      <!-- 제품별 거버넌스 현황 -->
      <section class="table-section">
        <h3 class="section-title" style="margin-bottom:16px">제품팀별 거버넌스 현황</h3>
        <div class="table-wrap">
          <table class="data-table">
            <thead>
              <tr>
                <th>제품</th>
                <th>AI 수</th>
                <th>규제 준수율</th>
                <th>조치 필요</th>
                <th>미해결 요청</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="p in productStats" :key="p.product" class="table-row" @click="onNavigate('inventory')">
                <td><span class="product-tag" :class="'product-' + p.product">{{ p.product }}</span></td>
                <td class="date-cell">{{ p.count }}개</td>
                <td>
                  <span class="score-value" :class="p.rate === 100 ? 'score-high' : 'score-low'">{{ p.rate }}%</span>
                </td>
                <td>
                  <span v-if="p.alertCount > 0" class="reason-tag">{{ p.alertCount }}건</span>
                  <span v-else class="comp-na">없음</span>
                </td>
                <td>
                  <span v-if="p.pendingCRs > 0" class="change-status badge-warning">{{ p.pendingCRs }}건</span>
                  <span v-else class="comp-na">없음</span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

    </div>

    <!-- 즉시 조치 필요 + 미해결 변경 요청 -->
    <div class="dashboard-bottom-grid">

      <!-- 즉시 조치 필요 -->
      <section class="table-section">
        <div class="table-header">
          <h3 class="section-title">즉시 조치 필요</h3>
          <button class="btn-link" @click="onNavigate('inventory')">
            전체 인벤토리
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
          </button>
        </div>
        <div v-if="alertServices.length === 0" class="success-banner">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
          <strong>조치가 필요한 서비스가 없습니다.</strong>
        </div>
        <div v-else class="table-wrap">
          <table class="data-table">
            <thead>
              <tr><th>AI명</th><th>제품</th><th>등급</th><th>조치 사유</th></tr>
            </thead>
            <tbody>
              <tr v-for="s in alertServices" :key="s.id" class="table-row" @click="onOpenDetail(s)">
                <td>
                  <div class="feature-cell">
                    <span class="feature-name">{{ s.feature }}</span>
                    <span class="feature-desc">{{ s.model }} {{ s.modelVersion }}</span>
                  </div>
                </td>
                <td><span class="product-tag-sm" :class="'product-' + s.product">{{ s.product }}</span></td>
                <td><span class="risk-badge" :class="getRiskClass(s.riskLevel)">{{ s.riskLevel }}</span></td>
                <td>
                  <div class="reason-cell">
                    <span v-for="r in getAlertReasons(s)" :key="r" class="reason-tag">{{ r }}</span>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- 최근 변경 이력 -->
      <section class="table-section">
        <div class="table-header">
          <h3 class="section-title">최근 변경 이력</h3>
        </div>
        <div class="table-wrap">
          <table class="data-table">
            <thead>
              <tr><th>날짜</th><th>제품</th><th>내용</th><th>상태</th></tr>
            </thead>
            <tbody>
              <tr v-for="(c, i) in recentChanges" :key="i" class="table-row" @click="() => { const s = findService(c); if (s) onOpenDetail(s) }">
                <td class="date-cell">{{ c.date }}</td>
                <td><span class="product-tag-sm" :class="'product-' + c.product">{{ c.product }}</span></td>
                <td>
                  <div class="feature-cell">
                    <span class="feature-name">{{ c.action }}</span>
                    <span class="feature-desc">{{ c.desc }}</span>
                  </div>
                </td>
                <td><span class="change-status" :class="getChangeStatusClass(c.status)">{{ c.status }}</span></td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

    </div>
  </div>
</template>
