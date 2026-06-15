<script setup>
import { ref } from 'vue'
import WorkflowView from './views/WorkflowView.vue'
import DashboardView from './views/DashboardView.vue'
import InventoryView from './views/InventoryView.vue'
import ComplianceView from './views/ComplianceView.vue'
import { aiServices, complianceItems } from './data/services.js'

const activeNav = ref('dashboard')
const showModal = ref(false)
const selectedService = ref(null)

const complianceViolationCount = aiServices.filter(s => Object.values(s.compliance).some(v => !v)).length
const alertCount = aiServices.filter(s => s.status === '주의' || s.status === '점검필요' || Object.values(s.compliance).some(v => !v)).length

function openDetail(service) { selectedService.value = service; showModal.value = true }
function closeModal() { showModal.value = false; selectedService.value = null }

function getScoreClass(score) { return score >= 90 ? 'score-high' : score >= 85 ? 'score-mid' : 'score-low' }
function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }
function getStatusClass(s) { return { '정상': 'status-ok', '주의': 'status-warn', '점검필요': 'status-danger' }[s] || '' }
function resolveIssue(issue) { alert(`이슈 조치 요청이 전달되었습니다.\n\n[${issue.type}] ${issue.desc}\n\n담당팀에 확인 요청이 접수되었습니다.`) }

const _C = { W: 480, H: 210, PL: 38, PR: 16, PT: 20, PB: 36, MIN: 70, MAX: 100 }
import { computed } from 'vue'
const qualityChartData = computed(() => {
  if (!selectedService.value) return null
  const history = selectedService.value.qualityHistory
  const labels = selectedService.value.qualityLabels
  const n = history.length
  const w = _C.W - _C.PL - _C.PR
  const h = _C.H - _C.PT - _C.PB
  const toX = i => _C.PL + (w / (n - 1)) * i
  const toY = v => _C.PT + (_C.MAX - v) / (_C.MAX - _C.MIN) * h
  const points = history.map((score, i) => ({ x: toX(i), y: toY(score), score, label: labels[i], valid: score > 0 }))
  const vp = points.filter(p => p.valid)
  const linePath = vp.map((p, i) => `${i === 0 ? 'M' : 'L'}${p.x},${p.y}`).join(' ')
  const areaPath = vp.length > 1
    ? `${linePath} L${vp[vp.length-1].x},${_C.H - _C.PB} L${vp[0].x},${_C.H - _C.PB}Z`
    : ''
  const gridLines = [70, 75, 80, 85, 90, 95, 100].map(v => ({ v, y: toY(v), isTh: v === 85 }))
  return { points, linePath, areaPath, gridLines, W: _C.W, H: _C.H, PL: _C.PL, PR: _C.PR, PT: _C.PT, PB: _C.PB }
})
</script>

<template>
  <div class="app-shell">
    <header class="header">
      <div class="header-inner">
        <div class="logo">
          <svg width="28" height="28" viewBox="0 0 28 28" fill="none"><rect width="28" height="28" rx="8" fill="#2BABEE"/><path d="M8 9h12M8 14h8M8 19h12" stroke="white" stroke-width="2.2" stroke-linecap="round"/></svg>
          <span class="logo-text">AI Governance Hub</span>
          <span class="logo-badge">DAOU</span>
        </div>
        <nav class="header-nav">
          <a :class="['nav-item', { active: activeNav === 'workflow' }]" @click="activeNav = 'workflow'">
            서비스 워크플로우
          </a>
          <a :class="['nav-item', { active: activeNav === 'dashboard' }]" @click="activeNav = 'dashboard'">
            대시보드
          </a>
          <a :class="['nav-item', { active: activeNav === 'inventory' }]" @click="activeNav = 'inventory'">
            AI 인벤토리
          </a>
          <a :class="['nav-item', { active: activeNav === 'compliance' }]" @click="activeNav = 'compliance'">
            규제 준수 현황
            <span v-if="complianceViolationCount > 0" class="nav-badge">{{ complianceViolationCount }}</span>
          </a>
        </nav>
        <div class="header-right">
          <div class="header-alert-wrap" v-if="alertCount > 0">
            <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/></svg>
            <span class="alert-badge">{{ alertCount }}</span>
          </div>
        </div>
      </div>
    </header>

    <main class="main">
      <WorkflowView v-if="activeNav === 'workflow'" />
      <DashboardView v-else-if="activeNav === 'dashboard'" :on-open-detail="openDetail" :on-navigate="nav => activeNav = nav" />
      <InventoryView v-else-if="activeNav === 'inventory'" :on-open-detail="openDetail" />
      <ComplianceView v-else-if="activeNav === 'compliance'" />

    </main>

    <!-- Detail Modal -->
    <Teleport to="body">
      <div v-if="showModal && selectedService" class="modal-overlay" @click.self="closeModal">
        <div class="modal">
          <div class="modal-header">
            <div>
              <div class="modal-title-row">
                <h3 class="modal-title">{{ selectedService.feature }}</h3>
                <span class="product-tag" :class="'product-' + selectedService.product">{{ selectedService.product }}</span>
                <span class="risk-badge" :class="getRiskClass(selectedService.riskLevel)">{{ selectedService.riskLevel }}</span>
                <span class="status-badge" :class="getStatusClass(selectedService.status)">{{ selectedService.status }}</span>
              </div>
              <p class="modal-desc">{{ selectedService.description }}</p>
            </div>
            <button class="modal-close" @click="closeModal">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
            </button>
          </div>

          <div class="modal-body">
            <!-- Basic Info -->
            <div class="info-grid">
              <div class="info-item"><span class="info-label">모델</span><span class="info-value">{{ selectedService.model }} {{ selectedService.modelVersion }}</span></div>
              <div class="info-item"><span class="info-label">담당팀</span><span class="info-value">{{ selectedService.owner }}</span></div>
              <div class="info-item"><span class="info-label">입력 데이터</span><span class="info-value">{{ selectedService.dataType }}</span></div>
              <div class="info-item"><span class="info-label">AI 분류</span><span class="info-value"><span class="risk-badge" :class="getRiskClass(selectedService.riskLevel)">{{ selectedService.riskLevel }}</span></span></div>
              <div class="info-item"><span class="info-label">학습 여부</span><span class="info-value">{{ selectedService.learningEnabled ? '⚠️ 학습 사용' : '✅ 비학습' }}</span></div>
              <div class="info-item"><span class="info-label">외부 API</span><span class="info-value">{{ selectedService.externalTransfer ? '⚠️ 외부 전송' : '✅ 내부 처리' }}</span></div>
            </div>

            <!-- Issues -->
            <div v-if="selectedService.issues.length > 0" class="modal-section modal-issue-section">
              <h4 class="modal-section-title modal-issue-title">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
                이슈
              </h4>
              <div class="modal-issue-list">
                <div v-for="(issue, i) in selectedService.issues" :key="i" class="modal-issue-item">
                  <div class="modal-issue-meta">
                    <span class="modal-issue-type">{{ issue.type }}</span>
                    <span class="modal-issue-date">{{ issue.date }}</span>
                  </div>
                  <p class="modal-issue-desc">{{ issue.desc }}</p>
                  <button class="btn-resolve" @click="resolveIssue(issue)">조치 요청</button>
                </div>
              </div>
            </div>

            <!-- Quality Chart -->
            <div class="modal-section">
              <h4 class="modal-section-title">품질 점수 추이</h4>
              <svg v-if="qualityChartData" class="quality-line-chart" :viewBox="`0 0 ${qualityChartData.W} ${qualityChartData.H}`">
                <defs>
                  <linearGradient id="qGrad" x1="0" y1="0" x2="0" y2="1">
                    <stop offset="0%" stop-color="#2BABEE" stop-opacity="0.18"/>
                    <stop offset="100%" stop-color="#2BABEE" stop-opacity="0"/>
                  </linearGradient>
                </defs>

                <!-- 가로 그리드선 + Y축 레이블 -->
                <g v-for="g in qualityChartData.gridLines" :key="g.v">
                  <line
                    :x1="qualityChartData.PL" :y1="g.y"
                    :x2="qualityChartData.W - qualityChartData.PR" :y2="g.y"
                    :stroke="g.isTh ? '#F44336' : '#E5E8EB'"
                    :stroke-width="g.isTh ? 1.5 : 1"
                    :stroke-dasharray="g.isTh ? '5 3' : null"
                    :opacity="g.isTh ? 0.9 : 1"
                  />
                  <text
                    :x="qualityChartData.PL - 6" :y="g.y + 4"
                    text-anchor="end" font-size="10" font-family="Pretendard,sans-serif"
                    :fill="g.isTh ? '#F44336' : '#ADB5BD'"
                    :font-weight="g.isTh ? '700' : '400'"
                  >{{ g.v }}</text>
                </g>

                <!-- 면적 채우기 -->
                <path v-if="qualityChartData.areaPath" :d="qualityChartData.areaPath" fill="url(#qGrad)"/>

                <!-- 꺾은선 -->
                <path :d="qualityChartData.linePath" fill="none" stroke="#2BABEE" stroke-width="2.5" stroke-linejoin="round" stroke-linecap="round"/>

                <!-- 데이터 포인트 + X축 레이블 -->
                <template v-for="p in qualityChartData.points" :key="p.label">
                  <template v-if="p.valid">
                    <text :x="p.x" :y="p.y - 10" text-anchor="middle" font-size="11" font-weight="700"
                      :fill="p.score < 85 ? '#F44336' : '#333D4B'" font-family="Pretendard,sans-serif">{{ p.score }}</text>
                    <circle :cx="p.x" :cy="p.y" r="5" :fill="p.score < 85 ? '#F44336' : '#2BABEE'" stroke="white" stroke-width="2.5"/>
                  </template>
                  <text :x="p.x" :y="qualityChartData.H - 8" text-anchor="middle"
                    font-size="11" fill="#868E96" font-family="Pretendard,sans-serif">{{ p.label }}</text>
                </template>
              </svg>
            </div>

            <!-- Compliance Checklist -->
            <div class="modal-section">
              <h4 class="modal-section-title">규제 준수 체크리스트</h4>
              <div class="compliance-list">
                <div v-for="item in complianceItems" :key="item.key" class="compliance-check-item" :class="selectedService.compliance[item.key] ? 'check-pass' : 'check-fail'">
                  <div class="check-icon">
                    <svg v-if="selectedService.compliance[item.key]" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
                    <svg v-else width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
                  </div>
                  <div class="check-body">
                    <span class="check-label">{{ item.label }}</span>
                    <span class="check-desc">{{ item.desc }}</span>
                  </div>
                  <span class="check-law">{{ item.law }}</span>
                </div>
              </div>
            </div>

            <!-- Prompt Versions -->
            <div class="modal-section">
              <h4 class="modal-section-title">프롬프트 변경 이력</h4>
              <div class="prompt-list">
                <div v-for="(p, i) in selectedService.promptVersions" :key="i" class="prompt-item">
                  <span class="prompt-version">{{ p.version }}</span>
                  <span class="prompt-date">{{ p.date }}</span>
                  <span class="prompt-change">{{ p.change }}</span>
                  <span class="prompt-author">{{ p.author }}</span>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn-secondary" @click="closeModal">닫기</button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>
