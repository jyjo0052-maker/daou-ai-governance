<script setup>
const actors = [
  {
    name: 'AI 운영부서',
    role: '서비스 개발 · 운영 · 이행',
    color: '#1B6B3A',
    tag: '각 서비스 개발팀',
    menus: [
      { title: '나의 서비스 관리', items: ['담당 AI 서비스 목록 조회', '신규 AI 등록 요청', '모델·프롬프트 변경 이력 작성', 'AI 분류별 분포', '서비스 중단·폐기 요청'] },
      { title: '품질 평가', items: ['서비스 유형별 품질 지표 정의 및 저장', '평가 결과 및 점수 추이'] },
      { title: 'AI 거버넌스 워크플로우', items: ['등록→평가→승인→운영→평가 단계별 진행 현황', '이슈 발생 시 알림 및 보고서 자동 생성'] },
    ],
  },
  {
    name: '법무팀',
    role: '법률 검토 · 규제 항목 관리',
    color: '#5B21B6',
    tag: '법무팀',
    menus: [
      { title: 'AI 영향 평가', items: ['신규 AI 등록 시 고영향 AI 판정 체크리스트', '평가 결과 기록 및 이력 보관', '판정 근거 문서 첨부 및 저장'] },
      { title: '규제 항목 관리', items: ['법률 개정 시 규제 세부 항목 변경', '변경 이력 관리', '서비스-규제 매핑'] },
      { title: '규제 준수 현황', items: ['의무 항목별 이행 현황 매트릭스', '서비스별 위반 항목 식별 및 조치 요청 발송'] },
    ],
  },
  {
    name: 'AX 기획팀',
    role: 'AI 거버넌스 총괄',
    color: '#0D47A1',
    tag: 'AX기획팀',
    ours: true,
    menus: [
      { title: '통합 대시보드', items: ['전사 AI 서비스 현황 KPI', '즉시 조치 필요 알림', '신규 AI 등록 요청 확인', '규제 준수 현황 요약', 'AI 분류별 분포', '최근 변경 이력 목록'] },
      { title: 'AI 인벤토리', items: ['전체 AI 서비스 목록', '실무 부서의 신규 AI 등록 요청 검토 및 승인', '모델·프롬프트 변경 이력 상세'] },
      { title: '규제 준수 현황', items: ['규제 항목별 세부 준수 현황', '규제 의무 항목 관리', '이슈 대응 관리'] },
    ],
  },
  {
    name: '경영진',
    role: '전략 방향 · 최종 의사결정',
    color: '#263238',
    tag: '경영진',
    menus: [
      { title: '요약 대시보드', items: ['전사 AI 위험 현황 요약', 'AI 유형별 분포', '월별 전사 평균 품질 점수 추이'] },
      { title: '규제 대응 현황 요약', items: ['이슈 대응 보고서 열람', '현재 적용 중인 규제 목록 및 준수율', '외부 감사 대응 리포트'] },
    ],
  },
]

const connectors = [
  { fwd: '등록 신청', back: '승인 / 반려' },
  { fwd: '검토 의견서', back: '검토 요청' },
  { fwd: '정기 보고', back: '전략 방향' },
]

const phases = [
  {
    num: '01', title: '신규 AI 서비스 등록', color: '#1B6B3A', bg: '#F0FBF4',
    steps: [
      { actor: 'AI 운영부서', color: '#1B6B3A', action: '신규 AI 서비스 기획서, 모델 정보, 데이터 처리 방식 등 등록 신청서 제출' },
      { actor: '법무팀', color: '#5B21B6', action: '고영향 AI 판정 체크리스트 완료, 적용 법조항 및 의무 이행 사항 검토 의견서 작성' },
      { actor: 'AX 기획팀', color: '#0D47A1', action: 'AI 유형 분류, 위험도 평가, 규제 검토 결과 종합 후 승인 / 반려 결정' },
      { actor: 'AI 운영부서', color: '#1B6B3A', action: '승인 시 AI 인벤토리 등록 후 운영 개시 / 반려 시 보완 후 재신청' },
    ],
  },
  {
    num: '02', title: '운영 중 규제 준수 관리', color: '#0D47A1', bg: '#EFF6FF',
    steps: [
      { actor: 'AI 운영부서', color: '#1B6B3A', action: '품질 점수 85점 이상 상시 유지, 이상 발생 시 AX기획팀 즉시 보고' },
      { actor: '법무팀', color: '#5B21B6', action: '법령 개정 반영, 규제 의무 항목 신설·수정 및 전 서비스 영향 검토' },
      { actor: 'AX 기획팀', color: '#0D47A1', action: '의무 항목별 이행 현황 모니터링, 미이행 서비스 조치 요청 및 기한 관리' },
      { actor: '경영진', color: '#263238', action: '분기별 AI 거버넌스 현황 보고 수신, 개선 방향 및 자원 배분 의사결정' },
    ],
  },
  {
    num: '03', title: '이슈 감지 및 대응', color: '#B71C1C', bg: '#FFF5F5',
    steps: [
      { actor: 'AX 기획팀', color: '#0D47A1', action: '정기 규제 점검으로 위반 이슈 감지, 심각도 평가 및 이슈 등록' },
      { actor: '법무팀', color: '#5B21B6', action: '이슈 법적 리스크 평가, 법조항 해석 및 조치 방향 가이드 제공' },
      { actor: 'AI 운영부서', color: '#1B6B3A', action: '조치 계획 수립 및 이행 (고지 문구 추가, UI 수정, 프로세스 개선 등)' },
      { actor: 'AX 기획팀', color: '#0D47A1', action: '조치 완료 검증 후 이슈 종결 / 심각도 높은 경우 경영진 에스컬레이션' },
    ],
  },
]
</script>

<template>
  <div class="wf-page">

    <!-- Hero -->
    <div class="wf-hero">
      <div>
        <div class="wf-eyebrow">DAOU Technology · AI Governance</div>
        <h2 class="wf-title">전체 서비스 워크플로우</h2>
        <p class="wf-sub">AI 기본법(2026.01.22 시행) 기반 다우기술 AI 거버넌스 운영 체계</p>
      </div>
      <div class="wf-stats">
        <div class="wf-stat"><span class="wf-stat-n">4</span><span class="wf-stat-l">참여 주체</span></div>
        <div class="wf-stat"><span class="wf-stat-n">3</span><span class="wf-stat-l">핵심 프로세스</span></div>
        <div class="wf-stat"><span class="wf-stat-n">6</span><span class="wf-stat-l">규제 의무 항목</span></div>
      </div>
    </div>

    <!-- 다이어그램 -->
    <section class="wf-card">
      <div class="wf-diagram">

        <!-- 액터 + 커넥터 한 줄 -->
        <div class="wf-row">
          <template v-for="(actor, idx) in actors" :key="actor.name">

            <!-- 액터 박스 -->
            <div class="wf-actor" :class="{ 'wf-actor-ours': actor.ours }" :style="{ borderTopColor: actor.color }">
              <div class="wf-ours-badge" v-if="actor.ours">우리 팀</div>
              <div class="wf-actor-hd">
                <div class="wf-icon" :style="{ background: actor.color }">
                  <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                    <template v-if="idx === 0"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8M12 17v4"/></template>
                    <template v-else-if="idx === 1"><path d="M12 2 2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5"/><path d="M2 12l10 5 10-5"/></template>
                    <template v-else-if="idx === 2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></template>
                    <template v-else><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></template>
                  </svg>
                </div>
                <div>
                  <div class="wf-actor-name">{{ actor.name }}</div>
                  <div class="wf-actor-role">{{ actor.role }}</div>
                </div>
              </div>
              <div class="wf-menus">
                <div v-for="m in actor.menus" :key="m.title" class="wf-menu">
                  <div class="wf-menu-title">{{ m.title }}</div>
                  <ul class="wf-menu-list">
                    <li v-for="it in m.items" :key="it">{{ it }}</li>
                  </ul>
                </div>
              </div>
              <div class="wf-actor-tag" :style="{ color: actor.color }">{{ actor.tag }}</div>
            </div>

            <!-- 커넥터 (마지막 아니면) -->
            <div v-if="idx < actors.length - 1" class="wf-conn">
              <span class="wf-conn-lbl wf-conn-fwd">
                <svg width="8" height="8" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><polyline points="5 12 19 12"/><polyline points="13 6 19 12 13 18"/></svg>
                {{ connectors[idx].fwd }}
              </span>
              <div class="wf-conn-line">
                <div class="wf-conn-track"></div>
                <div class="wf-conn-arrow"></div>
              </div>
              <span class="wf-conn-lbl wf-conn-back">
                {{ connectors[idx].back }}
                <svg width="8" height="8" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><polyline points="19 12 5 12"/><polyline points="11 6 5 12 11 18"/></svg>
              </span>
            </div>

          </template>
        </div>

        <!-- 피드백 화살표 -->
        <div class="wf-feedback">
          <div class="wf-fb-arrowhead"></div>
          <div class="wf-fb-label">
            <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/><line x1="12" y1="9" x2="12" y2="13"/></svg>
            이슈 조치 요청 · 운영 가이드라인 전달 · 모니터링 피드백 (AX 기획팀 → AI 운영부서)
          </div>
        </div>

      </div>
    </section>

    <!-- 프로세스 단계별 상세 -->
    <section class="wf-card">
      <h3 class="wf-phases-title">프로세스 단계별 상세</h3>
      <div class="wf-phases">
        <div v-for="p in phases" :key="p.num" class="wf-phase" :style="{ borderTopColor: p.color, background: p.bg }">
          <div class="wf-phase-hd">
            <span class="wf-phase-num" :style="{ background: p.color }">{{ p.num }}</span>
            <span class="wf-phase-name" :style="{ color: p.color }">{{ p.title }}</span>
          </div>
          <div class="wf-steps">
            <div v-for="(s, i) in p.steps" :key="i" class="wf-step">
              <div class="wf-step-n" :style="{ background: s.color }">{{ i + 1 }}</div>
              <div class="wf-step-body">
                <span class="wf-step-actor" :style="{ color: s.color }">{{ s.actor }}</span>
                <span class="wf-step-action">{{ s.action }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

  </div>
</template>

<style scoped>
.wf-page { display: flex; flex-direction: column; gap: 24px; padding-bottom: 40px; }

/* Hero */
.wf-hero { background: linear-gradient(135deg, #0D2137 0%, #1A3A5C 60%, #0D47A1 100%); border-radius: 12px; padding: 28px 32px; display: flex; align-items: center; justify-content: space-between; gap: 24px; flex-wrap: wrap; }
.wf-eyebrow { font-size: 10px; font-weight: 700; color: #90CAF9; letter-spacing: 0.12em; text-transform: uppercase; margin-bottom: 6px; }
.wf-title { font-size: 24px; font-weight: 800; color: white; margin: 0 0 6px; }
.wf-sub { font-size: 12px; color: #78909C; margin: 0; }
.wf-stats { display: flex; gap: 28px; flex-shrink: 0; }
.wf-stat { display: flex; flex-direction: column; align-items: center; gap: 3px; }
.wf-stat-n { font-size: 28px; font-weight: 800; color: white; line-height: 1; }
.wf-stat-l { font-size: 10px; color: #78909C; white-space: nowrap; }

/* Card */
.wf-card { background: white; border: 1px solid var(--gray-200); border-radius: 12px; padding: 24px; }

/* Diagram */
.wf-diagram { display: flex; flex-direction: column; gap: 0; }

/* 한 줄 행 */
.wf-row { display: grid; grid-template-columns: 1fr 60px 1fr 60px 1fr 60px 1fr; align-items: stretch; gap: 0; }

/* Actor */
.wf-actor { border-radius: 10px; padding: 14px; display: flex; flex-direction: column; gap: 8px; position: relative; border: 1.5px solid var(--gray-200); border-top-width: 3px; background: var(--gray-50); box-sizing: border-box; }
.wf-actor-ours { border-color: #2563EB; background: #EFF6FF; box-shadow: 0 0 0 3px #DBEAFE; }
.wf-ours-badge { position: absolute; top: -11px; left: 50%; transform: translateX(-50%); background: #0D47A1; color: white; font-size: 10px; font-weight: 700; padding: 2px 10px; border-radius: 10px; white-space: nowrap; }
.wf-actor-hd { display: flex; align-items: flex-start; gap: 8px; }
.wf-icon { width: 32px; height: 32px; border-radius: 7px; display: flex; align-items: center; justify-content: center; color: white; flex-shrink: 0; }
.wf-actor-name { font-size: 13px; font-weight: 800; color: var(--gray-900); line-height: 1.2; }
.wf-actor-role { font-size: 10px; font-weight: 600; color: var(--gray-500); margin-top: 2px; }
.wf-actor-tag { font-size: 10px; font-weight: 700; margin-top: auto; padding-top: 8px; border-top: 1px solid var(--gray-200); }

/* Menus */
.wf-menus { display: flex; flex-direction: column; gap: 6px; }
.wf-menu { background: white; border: 1px solid var(--gray-200); border-radius: 6px; padding: 7px 9px; }
.wf-menu-title { font-size: 10px; font-weight: 700; color: var(--gray-700); margin-bottom: 4px; }
.wf-menu-list { margin: 0; padding-left: 13px; display: flex; flex-direction: column; gap: 2px; }
.wf-menu-list li { font-size: 10px; color: var(--gray-500); line-height: 1.4; }

/* Connector */
.wf-conn { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 4px; padding: 0 2px; }
.wf-conn-lbl { display: flex; align-items: center; gap: 3px; font-size: 9px; font-weight: 600; white-space: nowrap; }
.wf-conn-fwd { color: #1E40AF; }
.wf-conn-back { color: var(--gray-400); }
.wf-conn-line { display: flex; align-items: center; width: 100%; }
.wf-conn-track { flex: 1; height: 2px; background: #BFDBFE; }
.wf-conn-arrow { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-left: 7px solid #93C5FD; }

/* Feedback arrow */
.wf-feedback { display: flex; align-items: center; gap: 8px; border: 2px dashed #FCA5A5; border-top: none; border-radius: 0 0 10px 10px; padding: 10px 16px; background: #FFF5F5; margin-top: 0; }
.wf-fb-arrowhead { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-right: 7px solid #EF4444; flex-shrink: 0; }
.wf-fb-label { display: flex; align-items: center; gap: 6px; font-size: 11px; font-weight: 600; color: #B91C1C; flex: 1; justify-content: center; }

/* Phases */
.wf-phases-title { font-size: 15px; font-weight: 700; color: var(--gray-800); margin: 0 0 18px; }
.wf-phases { display: grid; grid-template-columns: repeat(3, 1fr); gap: 14px; }
.wf-phase { border-radius: 10px; padding: 18px; border: 1px solid var(--gray-200); border-top-width: 3px; }
.wf-phase-hd { display: flex; align-items: center; gap: 8px; margin-bottom: 14px; }
.wf-phase-num { font-size: 10px; font-weight: 800; color: white; padding: 2px 7px; border-radius: 4px; }
.wf-phase-name { font-size: 13px; font-weight: 700; }
.wf-steps { display: flex; flex-direction: column; gap: 8px; }
.wf-step { display: flex; align-items: flex-start; gap: 8px; }
.wf-step-n { width: 18px; height: 18px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 10px; font-weight: 700; color: white; flex-shrink: 0; margin-top: 1px; }
.wf-step-body { display: flex; flex-direction: column; gap: 1px; }
.wf-step-actor { font-size: 10px; font-weight: 700; }
.wf-step-action { font-size: 11px; color: var(--gray-600); line-height: 1.5; }
</style>
