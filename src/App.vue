<script setup>
import { ref } from 'vue'
import DashboardView from './views/DashboardView.vue'
import InventoryView from './views/InventoryView.vue'
import ComplianceView from './views/ComplianceView.vue'
import OperationsView from './views/OperationsView.vue'
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
          <a :class="['nav-item', { active: activeNav === 'dashboard' }]" @click="activeNav = 'dashboard'">
            대시보드
          </a>
          <a :class="['nav-item', { active: activeNav === 'inventory' }]" @click="activeNav = 'inventory'">
            AI 인벤토리
          </a>
          <a :class="['nav-item', { active: activeNav === 'operations' }]" @click="activeNav = 'operations'">
            실무 운영 지원
          </a>
          <a :class="['nav-item', { active: activeNav === 'compliance' }]" @click="activeNav = 'compliance'">
            규제 준수
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
      <DashboardView v-if="activeNav === 'dashboard'" :on-open-detail="openDetail" :on-navigate="nav => activeNav = nav" />
      <InventoryView v-else-if="activeNav === 'inventory'" :on-open-detail="openDetail" />
      <OperationsView v-else-if="activeNav === 'operations'" :on-open-detail="openDetail" />
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
              <div class="info-item"><span class="info-label">생성형 AI</span><span class="info-value">{{ selectedService.isGenerativeAI ? '✅ 해당' : '❌ 비해당' }}</span></div>
              <div class="info-item"><span class="info-label">학습 여부</span><span class="info-value">{{ selectedService.learningEnabled ? '⚠️ 학습 사용' : '✅ 비학습' }}</span></div>
              <div class="info-item"><span class="info-label">외부 API</span><span class="info-value">{{ selectedService.externalTransfer ? '⚠️ 외부 전송' : '✅ 내부 처리' }}</span></div>
            </div>

            <!-- Quality Chart -->
            <div class="modal-section">
              <h4 class="modal-section-title">품질 점수 추이</h4>
              <div class="chart-area">
                <div class="chart-bars">
                  <div v-for="(score, idx) in selectedService.qualityHistory" :key="idx" class="chart-col">
                    <div class="chart-bar-wrap">
                      <span class="chart-val" v-if="score > 0">{{ score }}</span>
                      <div class="chart-bar" :class="getScoreClass(score)" :style="{ height: score > 0 ? (score * 0.9) + '%' : '0%' }"></div>
                    </div>
                    <span class="chart-label">{{ selectedService.qualityLabels[idx] }}</span>
                  </div>
                </div>
              </div>
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

            <!-- Issues -->
            <div class="modal-section" v-if="selectedService.issues.length > 0">
              <h4 class="modal-section-title">등록된 이슈</h4>
              <div class="issue-list">
                <div v-for="(issue, i) in selectedService.issues" :key="i" class="issue-item">
                  <span class="issue-type">{{ issue.type }}</span>
                  <span class="issue-date">{{ issue.date }}</span>
                  <span class="issue-desc">{{ issue.desc }}</span>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn-secondary" @click="closeModal">닫기</button>
            <button class="btn-primary-sm">변경 요청</button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>
