<script setup>
import { ref, computed } from 'vue'
import { aiServices, registrationRequests } from '../data/services.js'

const props = defineProps({ onOpenDetail: Function, initialTab: { type: String, default: '목록' } })

const activeTab = ref(props.initialTab)

// 서비스 목록 필터
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

const localRegs = ref([...registrationRequests])
const pendingRegs = computed(() => localRegs.value.filter(r => r.status === '검토 중').length)
function deleteReg(id) { localRegs.value = localRegs.value.filter(r => r.id !== id) }

// 등록 요청 검토 모달
const showRegModal = ref(false)
const selectedReg = ref(null)
function openRegModal(r) { selectedReg.value = r; showRegModal.value = true }
function closeRegModal() { showRegModal.value = false; selectedReg.value = null }
function approveReg() { alert(`[${selectedReg.value.id}] ${selectedReg.value.feature} 등록 요청을 승인했습니다.`); closeRegModal() }
function rejectReg() { alert(`[${selectedReg.value.id}] ${selectedReg.value.feature} 등록 요청을 반려했습니다.`); closeRegModal() }

function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }
function needsAction(s) {
  return s.qualityScore < QUALITY_THRESHOLD ||
    s.status === '주의' || s.status === '점검필요' ||
    Object.values(s.compliance).some(v => !v)
}
function getActionReasons(s) {
  const r = []
  if (s.qualityScore < QUALITY_THRESHOLD) r.push('품질 미달')
  if (s.status === '주의' || s.status === '점검필요') r.push(s.status)
  if (Object.values(s.compliance).some(v => !v)) r.push('규제 미준수')
  return r
}
function getStatusClass(s) { return { '정상': 'status-ok', '주의': 'status-warn', '점검필요': 'status-danger' }[s] || '' }
function getScoreClass(score) { return score >= 90 ? 'score-high' : score >= 85 ? 'score-mid' : 'score-low' }
function getComplianceStatus(s) {
  const vals = Object.values(s.compliance)
  const passed = vals.filter(Boolean).length
  if (passed === vals.length) return { label: '준수', cls: 'comp-ok' }
  if (passed >= vals.length - 1) return { label: '일부 미준수', cls: 'comp-warn' }
  return { label: '미준수', cls: 'comp-fail' }
}
function getRegSteps(reg) {
  const approved = reg.status === '승인 완료'
  return [
    {
      key: 'request', label: '요청 접수', initDone: true,
      items: [
        '운영부서 등록 신청서 및 서비스 기획서 수신',
        '서비스명·제품군·담당팀 기본 정보 확인',
        'AX기획팀 검토 착수 알림 및 접수 번호 발급',
      ],
    },
    {
      key: 'info', label: '정보 확인', initDone: true,
      items: [
        'AI 기능의 목적·활용 범위·대상 이용자 검토',
        '입력 데이터 유형·출처 및 외부 시스템 전송 여부 파악',
        '학습 데이터 포함 여부 및 데이터 보관 정책 확인',
        '기존 운영 서비스와의 중복·기능 충돌 여부 확인',
      ],
    },
    {
      key: 'risk', label: '위험도 평가', initDone: true,
      items: [
        'AI 기본법 제2조 기준 AI 유형 분류 (고영향 AI / 생성형 AI / 일반 AI)',
        '고영향 AI 해당 시: 생명·신체 안전·기본권 영향도 평가 및 인간 개입 가능성 점검',
        '생성형 AI 해당 시: 텍스트·이미지 등 생성물 표시 의무(제31조) 적용 범위 확인',
        '학습 데이터 저작권(TDM 면책 적용 불가) 및 개인정보 처리 적법성 검토',
        '위험 수준별 추가 준수 의무 목록 작성 및 담당팀 공유',
      ],
    },
    {
      key: 'legal', label: '규제 검토', initDone: true,
      items: [
        '투명성 고지 의무(제31조): 고영향·생성형 AI 사전 고지 문구 및 UI 반영 계획 검토',
        '위험관리 의무(제32조): 개발~폐기 전 생애주기 안전성 확보 방안 및 모니터링 체계 확인',
        '개인정보보호법: 동의·최소수집·목적 제한 원칙 및 가명·익명 처리 적용 여부',
        '이용자 권리 보장 절차(설명요구권·거부권·검토요구권) 마련 여부',
        '법무팀 법률 검토 의견서 수령 및 보완 요청 사항 반영',
      ],
    },
    {
      key: 'approve', label: '최종 승인', initDone: approved,
      items: [
        'AX기획팀 거버넌스 검토 결과 최종 확인 및 의사결정',
        'AI 인벤토리 등록 및 고유 관리 번호 부여',
        '운영부서 승인 통보 및 운영 가이드라인·체크리스트 전달',
        '품질 기준(85점 이상) 및 정기 모니터링 주기 수립',
      ],
    },
  ]
}
function getRegStatusClass(s) {
  return { '검토 중': 'reg-status-pending', '승인 완료': 'reg-status-approved', '반려': 'reg-status-rejected' }[s] || ''
}
</script>

<template>
  <div>
    <div class="view-header">
      <div>
        <h2 class="view-title">AI 인벤토리</h2>
        <p class="view-desc">다우기술 전체 AI 서비스 목록을 관리하고, 운영부서의 신규 AI 등록 요청을 검토·승인합니다.</p>
      </div>
    </div>

    <!-- 탭 -->
    <div class="inv-tab-bar">
      <button :class="['inv-tab', { active: activeTab === '목록' }]" @click="activeTab = '목록'">
        전체 AI 서비스 목록
      </button>
      <button :class="['inv-tab', { active: activeTab === '등록 요청' }]" @click="activeTab = '등록 요청'">
        신규 AI 등록 요청
        <span v-if="pendingRegs > 0" class="nav-badge">{{ pendingRegs }}</span>
      </button>
    </div>

    <!-- 탭: 전체 AI 서비스 목록 -->
    <template v-if="activeTab === '목록'">
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
                <th>AI 분류</th>
                <th>품질점수</th>
                <th>규제 준수</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="s in filteredServices" :key="s.id" @click="onOpenDetail(s)" :class="['table-row', { 'row-needs-action': needsAction(s) }]">
                <td><span class="product-tag" :class="'product-' + s.product">{{ s.product }}</span></td>
                <td>
                  <div class="feature-cell">
                    <div class="feature-name-row">
                      <span class="feature-name">{{ s.feature }}</span>
                      <span v-if="needsAction(s)" class="action-needed-tag">
                        <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
                        조치 필요
                      </span>
                    </div>
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
    </template>

    <!-- 탭: 신규 AI 등록 요청 -->
    <template v-else>
      <section class="table-section">
        <div class="table-header" style="margin-bottom:16px">
          <h3 class="section-title">신규 AI 등록 요청 검토</h3>
          <span class="result-count">총 {{ localRegs.length }}건 (검토 대기 {{ pendingRegs }}건)</span>
        </div>
        <p class="view-desc" style="margin-bottom:20px">운영부서가 신규 AI 서비스 도입 시 AX기획팀에 등록을 요청합니다. 위험등급 분류 및 규제 의무 사전 검토 후 승인합니다.</p>
        <div class="table-wrap">
          <table class="data-table">
            <thead>
              <tr>
                <th>요청 ID</th>
                <th>요청일</th>
                <th>제품</th>
                <th>AI 기능명</th>
                <th>등급</th>
                <th>요청 부서</th>
                <th>상태</th>
                <th>검토</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in localRegs" :key="r.id" class="table-row" @click="openRegModal(r)">
                <td class="date-cell" style="font-family:monospace;font-size:12px">{{ r.id }}</td>
                <td class="date-cell">{{ r.requestDate }}</td>
                <td><span class="product-tag" :class="'product-' + r.product">{{ r.product }}</span></td>
                <td><span class="feature-name">{{ r.feature }}</span></td>
                <td><span class="risk-badge" :class="getRiskClass(r.riskLevel)">{{ r.riskLevel }}</span></td>
                <td class="date-cell">{{ r.requestedBy }}</td>
                <td><span class="reg-status" :class="getRegStatusClass(r.status)">{{ r.status }}</span></td>
                <td>
                  <div style="display:flex;align-items:center;gap:8px">
                    <button v-if="r.status === '검토 중'" class="btn-action" @click.stop="openRegModal(r)">검토하기</button>
                    <span v-else class="comp-na">완료</span>
                    <button v-if="r.status !== '검토 중'" class="btn-icon-del" @click.stop="deleteReg(r.id)" title="목록에서 삭제">
                      <svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14H6L5 6"/><path d="M10 11v6M14 11v6"/><path d="M9 6V4h6v2"/></svg>
                    </button>
                  </div>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
    </template>
  </div>

  <!-- 등록 요청 검토 모달 -->
  <Teleport to="body">
    <div v-if="showRegModal && selectedReg" class="modal-overlay" @click.self="closeRegModal">
      <div class="modal">
        <div class="modal-header">
          <div>
            <div class="modal-title-row">
              <h3 class="modal-title">{{ selectedReg.feature }}</h3>
              <span class="product-tag" :class="'product-' + selectedReg.product">{{ selectedReg.product }}</span>
              <span class="risk-badge" :class="getRiskClass(selectedReg.riskLevel)">{{ selectedReg.riskLevel }}</span>
              <span class="reg-status reg-status-pending" style="font-size:12px;padding:3px 8px">검토 중</span>
            </div>
            <p class="modal-desc">신규 AI 등록 요청 — 위험등급 및 규제 의무 검토 후 승인 여부를 결정합니다.</p>
          </div>
          <button class="modal-close" @click="closeRegModal">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
          </button>
        </div>

        <div class="modal-body">
          <div class="modal-section">
            <h4 class="modal-section-title">요청 정보</h4>
            <div class="info-grid">
              <div class="info-item"><span class="info-label">요청일</span><span class="info-value">{{ selectedReg.requestDate }}</span></div>
              <div class="info-item"><span class="info-label">요청 부서</span><span class="info-value">{{ selectedReg.requestedBy }}</span></div>
              <div class="info-item"><span class="info-label">생성형 AI</span><span class="info-value">{{ selectedReg.isGenerativeAI ? '✅ 해당' : '❌ 비해당' }}</span></div>
            </div>
          </div>

          <div class="modal-section">
            <h4 class="modal-section-title">AI 상세 정보</h4>
            <div class="cr-detail-box" style="margin-bottom:14px">{{ selectedReg.description }}</div>
            <div class="info-grid">
              <div class="info-item" style="grid-column:1/-1">
                <span class="info-label">사용 모델</span>
                <span class="info-value">{{ selectedReg.model }}</span>
              </div>
              <div class="info-item" style="grid-column:1/-1">
                <span class="info-label">대상 이용자</span>
                <span class="info-value">{{ selectedReg.targetUsers }}</span>
              </div>
              <div class="info-item" style="grid-column:1/-1">
                <span class="info-label">출력 결과물</span>
                <span class="info-value">{{ selectedReg.outputType }}</span>
              </div>
              <div class="info-item" style="grid-column:1/-1">
                <span class="info-label">입력 데이터</span>
                <span class="info-value">{{ selectedReg.dataType }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">학습 데이터 사용</span>
                <span class="info-value">{{ selectedReg.learningEnabled ? '⚠️ 사용' : '✅ 미사용' }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">외부 API 호출</span>
                <span class="info-value">{{ selectedReg.externalAPI ? '⚠️ 있음' : '✅ 없음 (내부 처리)' }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">개인정보 처리</span>
                <span class="info-value">{{ selectedReg.personalData ? '⚠️ 포함 가능' : '✅ 해당 없음' }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">생성형 AI</span>
                <span class="info-value">{{ selectedReg.isGenerativeAI ? '✅ 해당' : '❌ 비해당' }}</span>
              </div>
            </div>
          </div>

          <div class="modal-section">
            <h4 class="modal-section-title">거버넌스 검토 진행 현황</h4>
            <div class="workflow-steps-v">
              <template v-for="(step, i) in getRegSteps(selectedReg)" :key="step.key">
                <div class="workflow-step-wrap">
                  <div class="workflow-step step-neutral">
                    <div class="step-head">
                      <div class="step-num" :class="step.initDone ? 'step-num-done' : 'step-num-pending'">
                        <svg v-if="step.initDone" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
                        <template v-else>{{ i + 1 }}</template>
                      </div>
                      <span class="step-label">{{ step.label }}</span>
                      <span v-if="step.initDone" class="step-done-badge">완료</span>
                    </div>
                    <ul class="step-items">
                      <li v-for="item in step.items" :key="item" class="step-item">
                        <span class="step-item-icon" :class="step.initDone ? 'step-icon-done' : 'step-icon-pending'">
                          <svg v-if="step.initDone" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
                          <svg v-else width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="9"/></svg>
                        </span>
                        <span class="step-check-text">{{ item }}</span>
                      </li>
                    </ul>
                  </div>
                </div>
                <div v-if="i < getRegSteps(selectedReg).length - 1" class="step-arrow-v">↓</div>
              </template>
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-secondary" @click="closeRegModal">닫기</button>
          <template v-if="selectedReg.status === '검토 중'">
            <button class="btn-danger-sm" @click="rejectReg">반려</button>
            <button class="btn-primary-sm" @click="approveReg">승인</button>
          </template>
        </div>
      </div>
    </div>
  </Teleport>
</template>
