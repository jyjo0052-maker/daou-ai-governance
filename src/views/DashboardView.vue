<script setup>
import { computed } from 'vue'
import { aiServices, complianceItems, registrationRequests } from '../data/services.js'

const props = defineProps({ onOpenDetail: Function, onNavigate: Function })

const QUALITY_THRESHOLD = 85

const totalServices = computed(() => aiServices.length)
const products = computed(() => [...new Set(aiServices.map(s => s.product))])

const complianceViolationCount = computed(() =>
  aiServices.filter(s => Object.values(s.compliance).some(v => !v)).length
)
const alertServices = computed(() =>
  aiServices.filter(s => s.status === '주의' || s.status === '점검필요' || Object.values(s.compliance).some(v => !v))
)
const belowThreshold = computed(() => aiServices.filter(s => s.qualityScore < QUALITY_THRESHOLD).length)

const pendingRegistrations = computed(() =>
  registrationRequests.filter(r => r.status === '검토 중')
)

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

const complianceItemStats = computed(() => {
  const stats = {}
  complianceItems.forEach(item => {
    const applicable = item.key === 'generativeAILabel'
      ? aiServices.filter(s => s.riskLevel === '생성형 AI' || s.riskLevel === '고영향 AI')
      : aiServices
    const pass = applicable.filter(s => s.compliance[item.key]).length
    stats[item.key] = {
      pass, total: applicable.length,
      rate: applicable.length ? Math.round(pass / applicable.length * 100) : 100,
    }
  })
  return stats
})

const DONUT_R = 70
const DONUT_COLORS = { '고영향 AI': '#F44336', '생성형 AI': '#2BABEE', '일반 AI': '#4CAF50' }
const donutSegments = computed(() => {
  const total = aiServices.length
  const C = 2 * Math.PI * DONUT_R
  let accumulated = 0
  return riskGroups.value.map(g => {
    const dash = (g.count / total) * C
    const seg = {
      level: g.level, count: g.count, rate: g.rate,
      color: DONUT_COLORS[g.level],
      dashArray: `${dash} ${C - dash}`,
      dashOffset: C * 0.25 - accumulated,
    }
    accumulated += dash
    return seg
  })
})

function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }
function getAlertReasons(s) {
  const r = []
  if (s.status === '주의' || s.status === '점검필요') r.push('품질 ' + s.status)
  const v = Object.values(s.compliance).filter(v => !v).length
  if (v > 0) r.push(`규제 미준수 ${v}건`)
  return r
}
</script>

<template>
  <div>
    <div class="view-header">
      <div>
        <h2 class="view-title">AI 거버넌스 현황</h2>
        <p class="view-desc">다우기술 전사 AI 서비스의 규제 준수·품질·등록 요청 현황을 통합 관리합니다.</p>
      </div>
      <span class="dashboard-date">기준일: 2026-06-15</span>
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

      <div class="kpi-card kpi-clickable" :class="alertServices.length > 0 ? 'kpi-alert' : 'kpi-good'" @click="onNavigate('inventory')">
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

      <div class="kpi-card kpi-clickable" :class="pendingRegistrations.length > 0 ? 'kpi-warn' : ''" @click="onNavigate('inventory', '등록 요청')">
        <div class="kpi-header">
          <span class="kpi-label">신규 등록 요청</span>
          <div class="kpi-icon" :class="pendingRegistrations.length > 0 ? 'kpi-icon-orange' : 'kpi-icon-blue'">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="16"/><line x1="8" y1="12" x2="16" y2="12"/></svg>
          </div>
        </div>
        <div class="kpi-value">{{ pendingRegistrations.length }}<span class="kpi-unit">건</span></div>
        <div class="kpi-sub">운영부서 신규 AI 등록 요청</div>
      </div>

    </section>

    <!-- 즉시 조치 필요 + 신규 AI 등록 요청 -->
    <div class="dashboard-bottom-grid" style="margin-bottom:24px">

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
              <tr><th>AI 기능</th><th>제품</th><th>등급</th><th>조치 사유</th></tr>
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

      <section class="table-section">
        <div class="table-header">
          <h3 class="section-title">신규 AI 등록 요청</h3>
          <button class="btn-link" @click="onNavigate('inventory', '등록 요청')">
            전체 보기
            <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
          </button>
        </div>
        <div v-if="pendingRegistrations.length === 0" class="success-banner">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
          <strong>검토 대기 중인 등록 요청이 없습니다.</strong>
        </div>
        <div v-else class="table-wrap">
          <table class="data-table">
            <thead>
              <tr><th>요청일</th><th>제품</th><th>AI 기능명</th><th>등급</th><th>상태</th></tr>
            </thead>
            <tbody>
              <tr v-for="r in pendingRegistrations" :key="r.id" class="table-row" @click="onNavigate('inventory', '등록 요청')">
                <td class="date-cell">{{ r.requestDate }}</td>
                <td><span class="product-tag-sm" :class="'product-' + r.product">{{ r.product }}</span></td>
                <td><span class="feature-name">{{ r.feature }}</span></td>
                <td><span class="risk-badge" :class="getRiskClass(r.riskLevel)">{{ r.riskLevel }}</span></td>
                <td><span class="reg-status reg-status-pending">검토 중</span></td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

    </div>

    <!-- AI 분류별 분포 + 제품별 거버넌스 -->
    <div class="dashboard-bottom-grid">

      <section class="table-section">
        <h3 class="section-title" style="margin-bottom:20px">AI 분류별 분포</h3>
        <div class="donut-wrap">
          <svg class="donut-svg" viewBox="0 0 200 200">
            <circle cx="100" cy="100" :r="DONUT_R" fill="none" stroke="#E5E8EB" stroke-width="30"/>
            <circle
              v-for="s in donutSegments" :key="s.level"
              cx="100" cy="100" :r="DONUT_R" fill="none"
              :stroke="s.color" stroke-width="30"
              :stroke-dasharray="s.dashArray"
              :stroke-dashoffset="s.dashOffset"
            />
            <text x="100" y="93" text-anchor="middle" class="donut-center-num">{{ totalServices }}</text>
            <text x="100" y="114" text-anchor="middle" class="donut-center-label">전체 서비스</text>
          </svg>
          <div class="donut-legend">
            <div v-for="s in donutSegments" :key="s.level" class="donut-legend-item">
              <span class="donut-dot" :style="{ background: s.color }"></span>
              <span class="donut-legend-name">{{ s.level }}</span>
              <span class="donut-legend-count">{{ s.count }}개</span>
              <span class="donut-legend-pct">{{ Math.round(s.count / totalServices * 100) }}%</span>
            </div>
          </div>
        </div>
      </section>

      <section class="table-section">
        <h3 class="section-title" style="margin-bottom:16px">규제 항목별 이행 현황</h3>
        <div class="comp-item-list">
          <div v-for="item in complianceItems" :key="item.key" class="comp-item-row">
            <span class="comp-item-law">{{ item.law }}</span>
            <span class="comp-item-label">{{ item.label }}</span>
            <span
              class="comp-item-count"
              :class="complianceItemStats[item.key].rate === 100 ? 'comp-item-ok' : 'comp-item-fail'"
            >
              <svg v-if="complianceItemStats[item.key].rate === 100" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
              <svg v-else width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
              {{ complianceItemStats[item.key].pass }}/{{ complianceItemStats[item.key].total }}개
            </span>
          </div>
        </div>
      </section>

    </div>
  </div>
</template>
