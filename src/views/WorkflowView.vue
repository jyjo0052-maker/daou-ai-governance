<template>
  <div class="wf-page">

    <!-- Hero -->
    <div class="wf-hero">
      <div>
        <div class="wf-eyebrow">DAOU Technology · AI Governance</div>
        <h2 class="wf-title">전체 서비스 워크플로우</h2>
        <p class="wf-sub">AI 기본법(2026.01.22 시행) 기반 다우기술 AI 거버넌스 운영 체계</p>
      </div>
    </div>

    <!-- 다이어그램 -->
    <section class="wf-card">
      <div class="wf-flow">

        <!-- AI 운영부서 -->
        <div class="wf-node" style="border-top-color:#1B6B3A">
          <div class="wf-node-icon" style="background:#1B6B3A">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8M12 17v4"/></svg>
          </div>
          <div class="wf-node-name">AI 운영부서</div>
          <div class="wf-node-desc">신규 AI 서비스 기획 및<br>등록 신청</div>
        </div>

        <!-- → 법무팀 -->
        <div class="wf-arrow">
          <div class="wf-arrow-label">신규 등록 신청</div>
          <div class="wf-arrow-line"><div class="wf-arrow-track"></div><div class="wf-arrow-head"></div></div>
        </div>

        <!-- 법무팀 -->
        <div class="wf-node" style="border-top-color:#5B21B6">
          <div class="wf-node-icon" style="background:#5B21B6">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2 2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5"/><path d="M2 12l10 5 10-5"/></svg>
          </div>
          <div class="wf-node-name">법무팀</div>
          <div class="wf-node-desc">고영향 AI 판정 및<br>법률 검토</div>
        </div>

        <!-- → AX 기획팀 -->
        <div class="wf-arrow">
          <div class="wf-arrow-label">검토 의견서 전달</div>
          <div class="wf-arrow-line"><div class="wf-arrow-track"></div><div class="wf-arrow-head"></div></div>
        </div>

        <!-- AX 기획팀 -->
        <div class="wf-node wf-node-ours" style="border-top-color:#0D47A1">
          <div class="wf-node-icon" style="background:#0D47A1">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          </div>
          <div class="wf-node-name">AX 기획팀</div>
          <div class="wf-node-desc">최종 승인 및<br>상시 규제 모니터링</div>
        </div>

        <!-- → 경영진 -->
        <div class="wf-arrow">
          <div class="wf-arrow-label">정기 보고</div>
          <div class="wf-arrow-line"><div class="wf-arrow-track"></div><div class="wf-arrow-head"></div></div>
        </div>

        <!-- 경영진 -->
        <div class="wf-node" style="border-top-color:#263238">
          <div class="wf-node-icon" style="background:#263238">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
          </div>
          <div class="wf-node-name">경영진</div>
          <div class="wf-node-desc">현황 파악 및<br>전략 방향 의사결정</div>
        </div>

      </div>
    </section>

  </div>
</template>

<style scoped>
.wf-page { display: flex; flex-direction: column; gap: 24px; padding-bottom: 40px; }

.wf-hero { background: linear-gradient(135deg, #0D2137 0%, #1A3A5C 60%, #0D47A1 100%); border-radius: 12px; padding: 28px 32px; }
.wf-eyebrow { font-size: 10px; font-weight: 700; color: #90CAF9; letter-spacing: 0.12em; text-transform: uppercase; margin-bottom: 6px; }
.wf-title { font-size: 24px; font-weight: 800; color: white; margin: 0 0 6px; }
.wf-sub { font-size: 12px; color: #78909C; margin: 0; }

.wf-card { background: white; border: 1px solid var(--gray-200); border-radius: 12px; padding: 40px 32px; }

/* Flow */
.wf-flow { display: flex; align-items: center; gap: 0; }

/* Node */
.wf-node { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 12px; padding: 28px 16px; border-radius: 12px; border: 1.5px solid var(--gray-200); border-top-width: 4px; background: var(--gray-50); text-align: center; }
.wf-node-ours { background: #EFF6FF; border-color: #2563EB; box-shadow: 0 0 0 3px #DBEAFE; }
.wf-node-icon { width: 48px; height: 48px; border-radius: 12px; display: flex; align-items: center; justify-content: center; color: white; }
.wf-node-name { font-size: 15px; font-weight: 800; color: var(--gray-900); }
.wf-node-desc { font-size: 12px; color: var(--gray-500); line-height: 1.6; }

/* Arrow */
.wf-arrow { display: flex; flex-direction: column; align-items: center; gap: 6px; flex-shrink: 0; width: 80px; }
.wf-arrow-label { font-size: 10px; font-weight: 600; color: #1E40AF; white-space: nowrap; }
.wf-arrow-line { display: flex; align-items: center; width: 100%; }
.wf-arrow-track { flex: 1; height: 2px; background: #BFDBFE; }
.wf-arrow-head { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-left: 8px solid #93C5FD; }
</style>
