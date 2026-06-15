<script setup>
import { ref, computed } from 'vue'
import { aiServices, complianceItems, complianceIssues } from '../data/services.js'

const regTab = ref('항목')
const complianceTab = ref('생성형 AI')
const expandedCompItem = ref(null)
const expandedRegItem = ref(null)
function toggleRegItem(key) { expandedRegItem.value = expandedRegItem.value === key ? null : key }

const regFilter = ref(null)
function setRegFilter(level) { regFilter.value = regFilter.value === level ? null : level; expandedRegItem.value = null }
const filteredRegItems = computed(() =>
  regFilter.value
    ? complianceItems.filter(item => isApplicable({ riskLevel: regFilter.value }, item.key))
    : complianceItems
)

const availableTabs = computed(() =>
  ['고영향 AI', '생성형 AI', '일반 AI'].filter(level => aiServices.some(s => s.riskLevel === level))
)
const tabServices = computed(() => aiServices.filter(s => s.riskLevel === complianceTab.value))
const tabRegulations = computed(() =>
  complianceItems.filter(item => isApplicable({ riskLevel: complianceTab.value }, item.key))
)
function switchTab(level) { complianceTab.value = level; expandedCompItem.value = null }
function toggleCompItem(key) { expandedCompItem.value = expandedCompItem.value === key ? null : key }

const tabSummary = computed(() => {
  const svcs = tabServices.value
  const regs = tabRegulations.value
  const fullyCompliant = regs.filter(item => svcs.every(s => s.compliance[item.key])).length
  const hasIssue = regs.filter(item => svcs.some(s => !s.compliance[item.key])).length
  const nonCompliantSvcs = svcs.filter(s => regs.some(item => !s.compliance[item.key])).length
  return { svcCount: svcs.length, regCount: regs.length, fullyCompliant, hasIssue, nonCompliantSvcs }
})

const showIssueModal = ref(false)
const selectedIssue = ref(null)
function openIssue(issue) { selectedIssue.value = issue; showIssueModal.value = true }
function closeIssue() { showIssueModal.value = false; selectedIssue.value = null }

const localIssues = ref([...complianceIssues])
function deleteIssue(id) { localIssues.value = localIssues.value.filter(i => i.id !== id) }

const itemMeta = {
  transparencyNotice: { lastModified: '2026-05-15', modifiedBy: '법무팀 김○○', historyCount: 3, target: '전체 서비스' },
  safetyAssessment:   { lastModified: '2026-03-10', modifiedBy: '법무팀 이○○', historyCount: 2, target: '전체 서비스' },
  dataProtection:     { lastModified: '2026-04-01', modifiedBy: '법무팀 박○○', historyCount: 2, target: '전체 서비스' },
  generativeAILabel:  { lastModified: '2026-02-20', modifiedBy: '법무팀 이○○', historyCount: 1, target: '생성형 AI · 고영향 AI' },
  incidentResponse:   { lastModified: '2026-01-22', modifiedBy: '법무팀 김○○', historyCount: 1, target: '전체 서비스' },
  regularReview:      { lastModified: '2026-01-22', modifiedBy: '법무팀 박○○', historyCount: 1, target: '전체 서비스' },
}

const changeHistory = [
  { date: '2026-05-15', item: '투명성 고지', change: '고영향 AI 서비스 고지 문구 기준 세분화', by: '법무팀 김○○', type: '수정' },
  { date: '2026-04-01', item: '개인정보 보호', change: '이용자 권리 보장 절차(설명요구권·거부권) 항목 추가', by: '법무팀 박○○', type: '수정' },
  { date: '2026-03-10', item: '투명성 고지', change: '적용 대상 생성형 AI 전체로 확대', by: '법무팀 이○○', type: '수정' },
  { date: '2026-02-20', item: 'AI 생성물 표시', change: '초기 항목 등록 (AI 기본법 제31조 적용)', by: '법무팀 이○○', type: '신규' },
  { date: '2026-01-22', item: '사고 대응 체계', change: '초기 항목 등록 (AI 기본법 제32조 적용)', by: '법무팀 김○○', type: '신규' },
  { date: '2026-01-22', item: '자동 모니터링', change: '내부 거버넌스 기준으로 초기 항목 등록', by: '법무팀 박○○', type: '신규' },
]

const mappingMatrix = [
  { label: '고영향 AI', keys: { transparencyNotice: true, safetyAssessment: true, dataProtection: true, generativeAILabel: true, incidentResponse: true, regularReview: true } },
  { label: '생성형 AI', keys: { transparencyNotice: true, safetyAssessment: true, dataProtection: true, generativeAILabel: true, incidentResponse: true, regularReview: true } },
  { label: '일반 AI',   keys: { transparencyNotice: true, safetyAssessment: true, dataProtection: true, generativeAILabel: false, incidentResponse: true, regularReview: true } },
]


function isApplicable(s, key) {
  if (key === 'generativeAILabel') return s.riskLevel === '생성형 AI' || s.riskLevel === '고영향 AI'
  return true
}


function getItemStatus(service, key) { return service.compliance[key] }
function getScoreClass(score) { return score === 100 ? 'score-high' : score >= 67 ? 'score-mid' : 'score-low' }
function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }



</script>

<template>
  <div>
    <!-- Header -->
    <div class="view-header">
      <div>
        <h2 class="view-title">규제 준수 현황</h2>
        <p class="view-desc">AI 기본법·개인정보보호법 기반 의무 이행 현황을 확인하고 외부 감사 대응 자료를 생성합니다.</p>
      </div>
    </div>

    <!-- 적용 법령 -->
    <div class="law-strip">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" style="color:var(--primary);flex-shrink:0"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
      <span class="law-strip-name">인공지능 발전과 신뢰 기반 조성 등에 관한 기본법</span>
      <span class="law-strip-divider">|</span>
      <span class="law-strip-meta">2026.01.22 시행</span>
      <span class="law-strip-divider">|</span>
      <span class="law-strip-meta">한국 최초 종합 AI 규제법</span>
    </div>

    <!-- 이슈 대응 관리 -->
    <section class="table-section" style="margin-bottom:24px">
      <div class="table-header" style="margin-bottom:16px">
        <h3 class="section-title">이슈 대응 관리</h3>
        <span class="result-count">
          총 {{ localIssues.length }}건
          (조치 중 {{ localIssues.filter(i => i.status === '조치 중').length }}건)
        </span>
      </div>
      <p class="view-desc" style="margin-bottom:20px">규제 의무 미이행으로 감지된 이슈의 조치 진행 현황을 추적합니다.</p>
      <div class="table-wrap">
        <table class="data-table">
          <thead>
            <tr>
              <th>이슈 ID</th>
              <th>서비스</th>
              <th>위반 항목</th>
              <th>내용</th>
              <th>감지일</th>
              <th>상태</th>
              <th>조치 완료일</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="issue in localIssues" :key="issue.id" class="table-row" @click="openIssue(issue)">
              <td class="date-cell" style="font-family:monospace;font-size:12px">{{ issue.id }}</td>
              <td>
                <div class="feature-cell">
                  <span class="feature-name">{{ issue.feature }}</span>
                  <span class="feature-desc">{{ issue.product }}</span>
                </div>
              </td>
              <td>
                <div class="feature-cell">
                  <span class="feature-name">{{ issue.item }}</span>
                  <span class="feature-desc">{{ issue.law }}</span>
                </div>
              </td>
              <td style="max-width:260px;font-size:13px;color:var(--gray-700)">{{ issue.description }}</td>
              <td class="date-cell">{{ issue.detectedDate }}</td>
              <td>
                <span class="issue-status-badge" :class="issue.status === '조치 완료' ? 'issue-status-done' : 'issue-status-ongoing'">
                  {{ issue.status }}
                </span>
              </td>
              <td class="date-cell">{{ issue.resolvedDate || '—' }}</td>
              <td>
                <button v-if="issue.status === '조치 완료'" class="btn-icon-del" @click.stop="deleteIssue(issue.id)" title="목록에서 삭제">
                  <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14H6L5 6"/><path d="M10 11v6M14 11v6"/><path d="M9 6V4h6v2"/></svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- 의무 항목별 준수 현황 -->
    <section class="table-section">
      <h3 class="section-title" style="margin-bottom:16px">의무 항목별 준수 현황</h3>

      <!-- 요약 -->
      <div class="comp-summary-bar">
        <div class="comp-summary-item">
          <span class="comp-summary-num">{{ tabSummary.svcCount }}</span>
          <span class="comp-summary-label">적용 서비스</span>
        </div>
        <div class="comp-summary-divider"></div>
        <div class="comp-summary-item">
          <span class="comp-summary-num">{{ tabSummary.regCount }}</span>
          <span class="comp-summary-label">의무 항목</span>
        </div>
        <div class="comp-summary-divider"></div>
        <div class="comp-summary-item">
          <span class="comp-summary-num" style="color:var(--success)">{{ tabSummary.fullyCompliant }}</span>
          <span class="comp-summary-label">전체 이행</span>
        </div>
        <div class="comp-summary-divider"></div>
        <div class="comp-summary-item">
          <span class="comp-summary-num" style="color:var(--danger)">{{ tabSummary.hasIssue }}</span>
          <span class="comp-summary-label">미이행 항목</span>
        </div>
        <div class="comp-summary-divider"></div>
        <div class="comp-summary-item">
          <span class="comp-summary-num" style="color:#E65100">{{ tabSummary.nonCompliantSvcs }}</span>
          <span class="comp-summary-label">조치 필요 서비스</span>
        </div>
      </div>

      <div class="inv-tab-bar" style="margin-bottom:20px">
        <button v-for="tab in availableTabs" :key="tab"
          :class="['inv-tab', { active: complianceTab === tab }]"
          @click="switchTab(tab)">
          <span class="risk-badge" :class="tab === '고영향 AI' ? 'risk-high' : tab === '생성형 AI' ? 'risk-generative' : 'risk-minimal'" style="font-size:10px;padding:1px 6px">{{ tab }}</span>
          {{ aiServices.filter(s => s.riskLevel === tab).length }}개 서비스
        </button>
      </div>

      <div class="comp-reg-list">
        <div v-for="item in tabRegulations" :key="item.key" class="comp-reg-card">
          <div class="comp-reg-header" @click="toggleCompItem(item.key)">
            <span class="reg-law-badge">{{ item.law }}</span>
            <div class="comp-reg-info">
              <span class="comp-reg-name">{{ item.label }}</span>
              <span class="comp-reg-desc">{{ item.desc }}</span>
            </div>
            <div class="comp-reg-stat">
              <span :class="tabServices.filter(s => !s.compliance[item.key]).length > 0 ? 'score-low' : 'score-high'"
                style="font-size:13px;font-weight:700">
                {{ tabServices.filter(s => s.compliance[item.key]).length }}/{{ tabServices.length }}개 이행
              </span>
            </div>
            <svg class="mapping-chevron" :class="{ open: expandedCompItem === item.key }"
              width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="6 9 12 15 18 9"/>
            </svg>
          </div>

          <div v-if="expandedCompItem === item.key" class="comp-reg-services">
            <div v-for="s in tabServices" :key="s.id" class="comp-reg-service-row">
              <span class="comp-svc-icon" :class="s.compliance[item.key] ? 'comp-svc-pass' : 'comp-svc-fail'">
                <svg v-if="s.compliance[item.key]" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
                <svg v-else width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
              </span>
              <span class="product-tag-sm" :class="'product-' + s.product">{{ s.product }}</span>
              <span class="mapping-service-name" style="flex:1">{{ s.feature }}</span>
              <span :class="s.compliance[item.key] ? 'issue-status-done' : 'issue-status-ongoing'"
                style="font-size:11px;font-weight:600;padding:2px 8px;border-radius:10px">
                {{ s.compliance[item.key] ? '이행' : '미이행' }}
              </span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- 규제 의무 항목 -->
    <section class="table-section" style="margin-bottom:24px">
      <div class="table-header" style="margin-bottom:16px">
        <h3 class="section-title">규제 의무 항목</h3>
        <span class="result-count">법무팀 관리 · 총 {{ complianceItems.length }}개 항목</span>
      </div>

      <div class="inv-tab-bar" style="margin-bottom:20px">
        <button :class="['inv-tab', { active: regTab === '항목' }]" @click="regTab = '항목'">규제 항목</button>
        <button :class="['inv-tab', { active: regTab === '이력' }]" @click="regTab = '이력'">변경 이력</button>
      </div>

      <!-- 탭 1: 규제 항목 -->
      <div v-if="regTab === '항목'">
        <div class="reg-filter-bar">
          <span class="reg-filter-label">AI 분류</span>
          <button v-for="level in ['고영향 AI', '생성형 AI', '일반 AI']" :key="level"
            :class="['reg-filter-btn', { active: regFilter === level }]"
            @click="setRegFilter(level)">
            <span class="risk-badge" :class="level === '고영향 AI' ? 'risk-high' : level === '생성형 AI' ? 'risk-generative' : 'risk-minimal'" style="font-size:10px;padding:1px 6px">{{ level }}</span>
          </button>
          <button v-if="regFilter" class="reg-filter-clear" @click="regFilter = null; expandedRegItem = null; ">전체 보기</button>
          <span class="reg-filter-count">{{ filteredRegItems.length }}개 항목</span>
        </div>
        <div class="reg-item-list">
        <div v-for="item in filteredRegItems" :key="item.key" class="reg-item-card">
          <div class="reg-item-top reg-item-top-click" @click="toggleRegItem(item.key)">
            <div class="reg-item-left">
              <span class="reg-law-badge">{{ item.law }}</span>
              <span class="reg-item-title">{{ item.label }}</span>
              <span class="reg-item-desc-inline">{{ item.desc }}</span>
            </div>
            <div style="display:flex;align-items:center;gap:12px;flex-shrink:0">
              <span class="reg-item-target">{{ item.target }}</span>
              <svg class="mapping-chevron" :class="{ open: expandedRegItem === item.key }"
                width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <polyline points="6 9 12 15 18 9"/>
              </svg>
            </div>
          </div>

          <div v-if="expandedRegItem === item.key" class="reg-item-detail">
            <div class="reg-item-legal-ref">{{ item.legalRef }}</div>

            <div class="reg-item-legal-box">
              <div class="reg-item-legal-label">
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>
                법령 원문 요약
              </div>
              <p class="reg-item-legal-text">{{ item.legalText }}</p>
            </div>

            <div class="reg-item-section-label">의무 이행 사항</div>
            <ul class="reg-item-obligations">
              <li v-for="(ob, i) in item.obligations" :key="i">{{ ob }}</li>
            </ul>

            <div class="reg-item-consequence">
              <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
              {{ item.consequence }}
            </div>
          </div>
        </div>
        </div>
      </div>

      <!-- 탭 2: 변경 이력 -->
      <div v-else-if="regTab === '이력'" class="table-wrap">
        <table class="data-table">
          <thead>
            <tr>
              <th>날짜</th>
              <th>항목</th>
              <th>변경 내용</th>
              <th>구분</th>
              <th>담당자</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="h in changeHistory" :key="h.date + h.item" class="table-row">
              <td class="date-cell">{{ h.date }}</td>
              <td><span class="feature-name">{{ h.item }}</span></td>
              <td style="font-size:13px;color:var(--gray-700);max-width:320px">{{ h.change }}</td>
              <td>
                <span class="history-type-badge" :class="h.type === '신규' ? 'hist-new' : 'hist-edit'">{{ h.type }}</span>
              </td>
              <td class="date-cell">{{ h.by }}</td>
            </tr>
          </tbody>
        </table>
      </div>

    </section>

  </div>

  <!-- 이슈 상세 모달 -->
  <Teleport to="body">
    <div v-if="showIssueModal && selectedIssue" class="modal-overlay" @click.self="closeIssue">
      <div class="modal modal-wide">
        <!-- 보고서 헤더 -->
        <div class="issue-report-header">
          <div class="issue-report-header-top">
            <div class="issue-report-id-row">
              <span class="issue-report-id">{{ selectedIssue.id }}</span>
              <span class="issue-status-badge" :class="selectedIssue.status === '조치 완료' ? 'issue-status-done' : 'issue-status-ongoing'">{{ selectedIssue.status }}</span>
              <span class="issue-severity-badge" :class="'severity-' + (selectedIssue.severity === '높음' ? 'high' : selectedIssue.severity === '중간' ? 'mid' : 'low')">위험도 {{ selectedIssue.severity }}</span>
            </div>
            <button class="modal-close" @click="closeIssue">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
            </button>
          </div>
          <h3 class="issue-report-title">{{ selectedIssue.feature }}</h3>
          <div class="issue-report-meta">
            <span class="product-tag" :class="'product-' + selectedIssue.product">{{ selectedIssue.product }}</span>
            <span class="reg-law-badge">{{ selectedIssue.law }}</span>
            <span class="issue-meta-item">
              <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
              감지 {{ selectedIssue.detectedDate }} · {{ selectedIssue.detectedBy }}
            </span>
            <span v-if="selectedIssue.resolvedDate" class="issue-meta-item">
              <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
              완료 {{ selectedIssue.resolvedDate }} · {{ selectedIssue.resolvedBy }}
            </span>
            <span v-else class="issue-meta-item">
              <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
              조치 기한 {{ selectedIssue.targetDate }}
            </span>
          </div>
        </div>

        <div class="modal-body">
          <!-- 이슈 내용 -->
          <div class="modal-section">
            <h4 class="modal-section-title">이슈 내용</h4>
            <div class="info-grid">
              <div class="info-item"><span class="info-label">위반 항목</span><span class="info-value">{{ selectedIssue.item }}</span></div>
              <div class="info-item"><span class="info-label">적용 법조항</span><span class="info-value">{{ selectedIssue.law }}</span></div>
              <div class="info-item" style="grid-column:1/-1"><span class="info-label">내용</span><span class="info-value">{{ selectedIssue.description }}</span></div>
            </div>
          </div>

          <!-- 팀별 조치 현황 -->
          <div class="modal-section">
            <h4 class="modal-section-title">팀별 조치 현황</h4>
            <div class="team-action-table">
              <div class="team-action-row team-action-head">
                <span>담당팀</span><span>조치 내용</span><span>완료 기한</span><span>상태</span>
              </div>
              <div v-for="(a, i) in selectedIssue.teamActions" :key="i" class="team-action-row">
                <span class="team-action-team">{{ a.team }}</span>
                <span class="team-action-desc">{{ a.action }}</span>
                <span class="team-action-date">{{ a.dueDate }}</span>
                <span class="team-action-status"
                  :class="a.status === '완료' ? 'ta-done' : a.status === '진행 중' ? 'ta-ongoing' : 'ta-pending'">
                  {{ a.status }}
                </span>
              </div>
            </div>
          </div>

          <!-- AI 분석 보고서 -->
          <div class="modal-section">
            <div class="air-wrap">
              <!-- 보고서 헤더 -->
              <div class="air-top">
                <div class="air-title-row">
                  <span class="air-icon">
                    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8M12 17v4"/></svg>
                  </span>
                  <span class="air-title">이슈 분석 AI 보고서</span>
                  <span class="air-badge">AI 자동 생성</span>
                </div>
                <div class="air-meta-row">
                  <span class="air-meta-item">생성 {{ selectedIssue.reportGeneratedAt }}</span>
                  <span class="air-meta-sep">·</span>
                  <span class="air-meta-item">{{ selectedIssue.reportModel }}</span>
                  <span class="air-meta-sep">·</span>
                  <span class="air-meta-item">규제 위반 이슈 분석 v1</span>
                </div>
              </div>

              <!-- 종합 위험도 -->
              <div class="air-risk-bar">
                <span class="air-risk-label">종합 위험도</span>
                <div class="air-risk-track">
                  <div class="air-risk-fill" :style="{ width: selectedIssue.riskScore * 10 + '%', background: selectedIssue.riskScore >= 7 ? 'var(--danger)' : selectedIssue.riskScore >= 4 ? '#FF8F00' : 'var(--success)' }"></div>
                </div>
                <span class="air-risk-score" :style="{ color: selectedIssue.riskScore >= 7 ? 'var(--danger)' : selectedIssue.riskScore >= 4 ? '#FF8F00' : 'var(--success)' }">
                  {{ selectedIssue.riskScore }}/10
                </span>
              </div>

              <!-- 1. 종합 요약 -->
              <div class="air-section">
                <div class="air-section-label">
                  <span class="air-section-num">01</span> 종합 요약
                </div>
                <p class="air-section-text">{{ selectedIssue.summary }}</p>
              </div>

              <!-- 2. 원인 분석 -->
              <div class="air-section">
                <div class="air-section-label">
                  <span class="air-section-num">02</span> 원인 분석
                </div>
                <div class="air-cause-list">
                  <div v-for="(c, i) in selectedIssue.rootCauseAnalysis" :key="i" class="air-cause-item">
                    <span class="air-cause-cat">{{ c.category }}</span>
                    <p class="air-cause-text">{{ c.detail }}</p>
                  </div>
                </div>
              </div>

              <!-- 3. 영향 및 리스크 평가 -->
              <div class="air-section">
                <div class="air-section-label">
                  <span class="air-section-num">03</span> 영향 및 리스크 평가
                </div>
                <p class="air-section-text">{{ selectedIssue.impactDetail }}</p>
              </div>

              <!-- 4. 권고사항 -->
              <div class="air-section">
                <div class="air-section-label">
                  <span class="air-section-num">04</span> 권고사항
                </div>
                <div class="air-rec-list">
                  <div v-for="(r, i) in selectedIssue.recommendations" :key="i" class="air-rec-item">
                    <div class="air-rec-top">
                      <span class="air-rec-priority" :class="r.priority === '긴급' ? 'rec-urgent' : r.priority === '높음' ? 'rec-high' : r.priority === '중간' ? 'rec-mid' : 'rec-done'">
                        {{ r.priority }}
                      </span>
                      <span class="air-rec-timeframe">{{ r.timeframe }}</span>
                    </div>
                    <p class="air-rec-action">{{ r.action }}</p>
                    <span class="air-rec-owner">담당: {{ r.owner }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-secondary" @click="closeIssue">닫기</button>
        </div>
      </div>
    </div>
  </Teleport>
</template>
