<script setup>
import { ref, computed } from 'vue'
import { aiServices } from '../data/services.js'

const QUALITY_THRESHOLD = ref(85)
const selectedService = ref(aiServices[0])

const workflowSteps = [
  { key: 'register', label: '등록', desc: 'AI 서비스 등록 및 기본 정보 입력' },
  { key: 'evaluate', label: '평가', desc: '품질 지표 설정 및 초기 평가 수행' },
  { key: 'approve', label: '승인', desc: '법무·경영진 검토 및 도입 승인' },
  { key: 'operate', label: '운영', desc: 'AI 서비스 배포 및 모니터링 시작' },
  { key: 'review', label: '자동 모니터링', desc: '임계치 기반 실시간 품질·안전성 자동 모니터링 가동' },
]

const props = defineProps({ onOpenDetail: Function })

const belowThreshold = computed(() => aiServices.filter(s => s.qualityScore < QUALITY_THRESHOLD.value))

// 개선 요청 모달
const showImprovementModal = ref(false)
const improvementTarget = ref(null)
const improvementForm = ref({ desc: '', requester: '' })

function openImprovementModal(service) {
  improvementTarget.value = service
  improvementForm.value = { desc: '', requester: '' }
  showImprovementModal.value = true
}
function closeImprovementModal() { showImprovementModal.value = false; improvementTarget.value = null }
function submitImprovement() {
  alert(`개선 요청이 제출되었습니다.\n서비스: ${improvementTarget.value.feature}\n유형: ${improvementForm.value.type}\n요청자: ${improvementForm.value.requester}`)
  closeImprovementModal()
}

function getStepClass(step) {
  if (!selectedService.value) return 'step-pending'
  return selectedService.value.workflowStatus[step.key] ? 'step-done' : 'step-pending'
}

function getRiskClass(l) { return { '고영향 AI': 'risk-high', '생성형 AI': 'risk-generative', '일반 AI': 'risk-minimal' }[l] || '' }
function getScoreClass(score) { return score >= 90 ? 'score-high' : score >= 85 ? 'score-mid' : 'score-low' }
</script>

<template>
  <div>
    <div class="view-header">
      <div>
        <h2 class="view-title">실무 운영 지원</h2>
        <p class="view-desc">AI 거버넌스 프로세스를 절차대로 수행하고, 품질 임계치 기반으로 즉시 조치가 필요한 서비스를 확인합니다.</p>
      </div>
    </div>

    <!-- Top Two-Column Layout -->
    <div class="dashboard-bottom-grid" style="margin-bottom:24px">

      <!-- 품질 점검 필요 -->
      <section class="table-section">
        <div class="threshold-header">
          <h3 class="section-title">품질 점검 필요</h3>
          <div class="threshold-control">
            <span class="threshold-label">임계치:</span>
            <input type="number" v-model="QUALITY_THRESHOLD" min="0" max="100" class="threshold-input" />
            <span class="threshold-unit">점</span>
          </div>
        </div>

        <div v-if="belowThreshold.length > 0" class="alert-banner">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
          <strong>임계치({{ QUALITY_THRESHOLD }}점) 미달 {{ belowThreshold.length }}건 — 즉시 검토 필요</strong>
        </div>
        <div v-else class="success-banner">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><path d="M22 4 12 14.01l-3-3"/></svg>
          <strong>모든 서비스가 임계치({{ QUALITY_THRESHOLD }}점) 이상입니다.</strong>
        </div>

        <div v-if="belowThreshold.length > 0" class="table-wrap" style="margin-top:16px">
          <table class="data-table">
            <thead>
              <tr><th>제품</th><th>AI 기능</th><th>품질점수</th><th>필요 조치</th></tr>
            </thead>
            <tbody>
              <tr v-for="s in belowThreshold" :key="s.id" class="table-row" @click="onOpenDetail(s)">
                <td><span class="product-tag" :class="'product-' + s.product">{{ s.product }}</span></td>
                <td><span class="feature-name">{{ s.feature }}</span></td>
                <td><span class="score-value score-low">{{ s.qualityScore }}점</span></td>
                <td><button class="btn-action" @click.stop="openImprovementModal(s)">개선 요청</button></td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

    </div>

    <!-- Workflow -->
    <section class="table-section" style="margin-bottom:24px">
      <h3 class="section-title" style="margin-bottom:16px">AI 도입·운영 절차 (거버넌스 워크플로우)</h3>
      <p class="view-desc" style="margin-bottom:20px">이 절차를 따르면 AI 기본법 위험관리 의무(제32조)를 자동으로 준수할 수 있습니다.</p>

      <!-- Step Selector -->
      <div class="service-selector">
        <label class="filter-label">서비스 선택</label>
        <select v-model="selectedService" class="filter-select">
          <option v-for="s in aiServices" :key="s.id" :value="s">{{ s.product }} — {{ s.feature }}</option>
        </select>
      </div>

      <!-- Workflow Steps -->
      <div class="workflow-steps">
        <div v-for="(step, idx) in workflowSteps" :key="step.key" class="workflow-step-wrap">
          <div class="workflow-step" :class="getStepClass(step)">
            <div class="step-num">
              <svg v-if="getStepClass(step) === 'step-done'" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>
              <span v-else>{{ idx + 1 }}</span>
            </div>
            <div class="step-body">
              <span class="step-label">{{ step.label }}</span>
              <span class="step-desc">{{ step.desc }}</span>
            </div>
            <span class="step-status-tag" :class="getStepClass(step) === 'step-done' ? 'step-tag-done' : 'step-tag-pending'">
              {{ getStepClass(step) === 'step-done' ? '완료' : '미완료' }}
            </span>
          </div>
          <div v-if="idx < workflowSteps.length - 1" class="step-arrow">→</div>
        </div>
      </div>
    </section>

    <!-- Quality Evaluation Guide -->
    <section class="table-section">
      <h3 class="section-title" style="margin-bottom:8px">품질 평가 기준</h3>
      <p class="view-desc" style="margin-bottom:20px">각 서비스 유형에 맞는 평가 지표와 계산 방식을 제공합니다.</p>
      <div class="eval-grid">
        <div class="eval-card">
          <div class="eval-title">생성형 AI (메일 요약·문자 생성)</div>
          <div class="eval-items">
            <div class="eval-item"><span class="eval-key">간결성</span><span class="eval-val">원문 대비 요약 비율 ≤ 30%</span></div>
            <div class="eval-item"><span class="eval-key">정확성</span><span class="eval-val">핵심 정보 누락 없음</span></div>
            <div class="eval-item"><span class="eval-key">정보성</span><span class="eval-val">주요 수치·날짜 포함</span></div>
            <div class="eval-item"><span class="eval-key">가독성</span><span class="eval-val">문단 구조 명확</span></div>
          </div>
        </div>
        <div class="eval-card">
          <div class="eval-title">분류·추천 AI (카테고리·검색어)</div>
          <div class="eval-items">
            <div class="eval-item"><span class="eval-key">정확도</span><span class="eval-val">Top-1 정확도 ≥ 85%</span></div>
            <div class="eval-item"><span class="eval-key">커버리지</span><span class="eval-val">전체 상품 대비 추천율</span></div>
            <div class="eval-item"><span class="eval-key">일관성</span><span class="eval-val">동일 입력 동일 출력</span></div>
            <div class="eval-item"><span class="eval-key">지연시간</span><span class="eval-val">응답 3초 이내</span></div>
          </div>
        </div>
        <div class="eval-card">
          <div class="eval-title">OCR·문서 인식 AI</div>
          <div class="eval-items">
            <div class="eval-item"><span class="eval-key">인식률</span><span class="eval-val">94% 이상 (오류 제외 98%)</span></div>
            <div class="eval-item"><span class="eval-key">필드 정확도</span><span class="eval-val">핵심 필드 100% 추출</span></div>
            <div class="eval-item"><span class="eval-key">보안</span><span class="eval-val">개인정보 마스킹 여부</span></div>
            <div class="eval-item"><span class="eval-key">처리속도</span><span class="eval-val">문서당 10초 이내</span></div>
          </div>
        </div>
      </div>
    </section>
  </div>

  <!-- 개선 요청 모달 -->
  <Teleport to="body">
    <div v-if="showImprovementModal && improvementTarget" class="modal-overlay" @click.self="closeImprovementModal">
      <div class="modal">
        <div class="modal-header">
          <div>
            <div class="modal-title-row">
              <h3 class="modal-title">{{ improvementTarget.feature }}</h3>
              <span class="product-tag" :class="'product-' + improvementTarget.product">{{ improvementTarget.product }}</span>
              <span class="risk-badge" :class="getRiskClass(improvementTarget.riskLevel)">{{ improvementTarget.riskLevel }}</span>
            </div>
            <p class="modal-desc">품질 개선 요청 — 담당팀에 검토 요청을 전달합니다.</p>
          </div>
          <button class="modal-close" @click="closeImprovementModal">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
          </button>
        </div>

        <div class="modal-body">
          <!-- 현재 품질 현황 -->
          <div class="modal-section">
            <h4 class="modal-section-title">현재 품질 현황</h4>
            <div class="info-grid">
              <div class="info-item">
                <span class="info-label">현재 점수</span>
                <span class="info-value"><span class="score-value score-low" style="font-size:20px;font-weight:700">{{ improvementTarget.qualityScore }}점</span></span>
              </div>
              <div class="info-item">
                <span class="info-label">임계치</span>
                <span class="info-value">{{ QUALITY_THRESHOLD }}점 (미달 {{ QUALITY_THRESHOLD - improvementTarget.qualityScore }}점)</span>
              </div>
              <div class="info-item">
                <span class="info-label">모델</span>
                <span class="info-value">{{ improvementTarget.model }} {{ improvementTarget.modelVersion }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">담당팀</span>
                <span class="info-value">{{ improvementTarget.owner }}</span>
              </div>
            </div>
            <div class="chart-area" style="margin-top:16px">
              <div class="chart-bars">
                <div v-for="(score, idx) in improvementTarget.qualityHistory" :key="idx" class="chart-col">
                  <div class="chart-bar-wrap">
                    <span class="chart-val" v-if="score > 0">{{ score }}</span>
                    <div class="chart-bar" :class="getScoreClass(score)" :style="{ height: score > 0 ? (score * 0.9) + '%' : '0%' }"></div>
                  </div>
                  <span class="chart-label">{{ improvementTarget.qualityLabels[idx] }}</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 개선 요청 양식 -->
          <div class="modal-section">
            <h4 class="modal-section-title">검토 요청</h4>
            <p class="view-desc" style="margin-bottom:16px">품질 점수가 임계치 미달로 자동 감지됐습니다. 담당팀에 확인을 요청합니다.</p>
            <div class="improvement-form">
              <div class="form-row">
                <label class="form-label">요청자</label>
                <input v-model="improvementForm.requester" type="text" class="filter-input" placeholder="이름 입력" style="flex:1" />
              </div>
              <div class="form-row form-row-col">
                <label class="form-label">추가 메모</label>
                <textarea v-model="improvementForm.desc" class="improvement-textarea" placeholder="특이사항이나 추가로 전달할 내용이 있으면 입력하세요. (선택)" rows="3"></textarea>
              </div>
            </div>
          </div>
        </div>

        <div class="modal-footer">
          <button class="btn-secondary" @click="closeImprovementModal">취소</button>
          <button class="btn-primary-sm" @click="submitImprovement" :disabled="!improvementForm.requester">검토 요청 전송</button>
        </div>
      </div>
    </div>
  </Teleport>
</template>
