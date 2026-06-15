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
      <div class="wf-diagram">

        <!-- ── Row 1: nodes + connectors ── -->

        <!-- AI 운영부서 -->
        <div class="wf-node" style="border-top-color:#1B6B3A">
          <div class="wf-node-icon" style="background:#1B6B3A">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8M12 17v4"/></svg>
          </div>
          <div class="wf-node-name">AI 운영부서</div>
          <div class="wf-node-desc">신규 AI 서비스 기획 및<br>등록 신청</div>
        </div>

        <!-- → 법무팀 -->
        <div class="wf-conn">
          <div class="wf-conn-lbl wf-fwd">신규 등록 신청</div>
          <div class="wf-line"><div class="wf-track"></div><div class="wf-head"></div></div>
        </div>

        <!-- 법무팀 -->
        <div class="wf-node" style="border-top-color:#5B21B6">
          <div class="wf-node-icon" style="background:#5B21B6">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2 2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5"/><path d="M2 12l10 5 10-5"/></svg>
          </div>
          <div class="wf-node-name">법무팀</div>
          <div class="wf-node-desc">고영향 AI 판정 및<br>법률 검토 · 규제 항목 관리</div>
        </div>

        <!-- → AX기획팀 -->
        <div class="wf-conn">
          <div class="wf-conn-lbl wf-fwd">검토 의견서 전달</div>
          <div class="wf-line"><div class="wf-track"></div><div class="wf-head"></div></div>
        </div>

        <!-- AX 기획팀 -->
        <div class="wf-node wf-node-ours" style="border-top-color:#0D47A1">
          <div class="wf-node-icon" style="background:#0D47A1">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          </div>
          <div class="wf-node-name">AX 기획팀</div>
          <div class="wf-node-desc">최종 승인 및<br>상시 규제 모니터링</div>
        </div>

        <!-- ↔ 경영진 (양방향) -->
        <div class="wf-conn wf-conn-bi">
          <div class="wf-conn-lbl wf-fwd">
            <svg width="8" height="8" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="5" y1="12" x2="19" y2="12"/><polyline points="13 6 19 12 13 18"/></svg>
            정기 보고
          </div>
          <div class="wf-line"><div class="wf-track"></div><div class="wf-head"></div></div>
          <div class="wf-conn-lbl wf-back">
            전략 방향
            <svg width="8" height="8" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="19" y1="12" x2="5" y2="12"/><polyline points="11 6 5 12 11 18"/></svg>
          </div>
        </div>

        <!-- 경영진 -->
        <div class="wf-node" style="border-top-color:#263238">
          <div class="wf-node-icon" style="background:#263238">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
          </div>
          <div class="wf-node-name">경영진</div>
          <div class="wf-node-desc">현황 파악 및<br>전략 방향 의사결정</div>
        </div>

        <!-- ── Row 2: 승인/반려 복귀 화살표 (col 1–5) ── -->
        <div class="wf-return">
          <div class="wf-rhead"></div>
          <div class="wf-rtrack"></div>
          <div class="wf-rlbl">
            <svg width="10" height="10" viewBox="0 0 24 24" fill="none" stroke="#E53E3E" stroke-width="2.5"><polyline points="9 14 4 9 9 4"/><path d="M20 20v-7a4 4 0 0 0-4-4H4"/></svg>
            승인 / 반려 통보
          </div>
          <div class="wf-rtrack"></div>
        </div>

        <!-- col 6–7 row 2: empty -->
        <div></div>
        <div></div>

      </div>
    </section>

  </div>
</template>

<style scoped>
.wf-page { display: flex; flex-direction: column; gap: 24px; padding-bottom: 40px; }

.wf-hero {
  background: linear-gradient(135deg, #0D2137 0%, #1A3A5C 60%, #0D47A1 100%);
  border-radius: 12px; padding: 28px 32px;
}
.wf-eyebrow { font-size: 10px; font-weight: 700; color: #90CAF9; letter-spacing: 0.12em; text-transform: uppercase; margin-bottom: 6px; }
.wf-title { font-size: 24px; font-weight: 800; color: white; margin: 0 0 6px; }
.wf-sub { font-size: 12px; color: #78909C; margin: 0; }

.wf-card { background: white; border: 1px solid var(--gray-200); border-radius: 12px; padding: 36px 28px 28px; }

/* Unified grid */
.wf-diagram {
  display: grid;
  grid-template-columns: 1fr 80px 1fr 80px 1fr 96px 1fr;
  grid-template-rows: auto auto;
  align-items: center;
  row-gap: 0;
}

/* Row 1 items center vertically */
.wf-node, .wf-conn { align-self: center; }

/* Node */
.wf-node {
  display: flex; flex-direction: column; align-items: center; gap: 10px;
  padding: 24px 12px; border-radius: 12px;
  border: 1.5px solid var(--gray-200); border-top-width: 4px;
  background: var(--gray-50); text-align: center;
}
.wf-node-ours { background: #EFF6FF; border-color: #2563EB; box-shadow: 0 0 0 3px #DBEAFE; }
.wf-node-icon { width: 44px; height: 44px; border-radius: 11px; display: flex; align-items: center; justify-content: center; color: white; }
.wf-node-name { font-size: 14px; font-weight: 800; color: var(--gray-900); }
.wf-node-desc { font-size: 11px; color: var(--gray-500); line-height: 1.6; }

/* Connector */
.wf-conn { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 5px; padding: 0 4px; }
.wf-conn-lbl { display: flex; align-items: center; gap: 3px; font-size: 9px; font-weight: 600; white-space: nowrap; }
.wf-fwd  { color: #1E40AF; }
.wf-back { color: var(--gray-500); }
.wf-line { display: flex; align-items: center; width: 100%; }
.wf-track { flex: 1; height: 2px; background: #BFDBFE; }
.wf-head { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-left: 8px solid #93C5FD; }

/* Return arrow row: col 1–5, row 2 */
.wf-return {
  grid-column: 1 / 6;
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 14px 0 4px;
}
/* arrowhead pointing left */
.wf-rhead { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-right: 8px solid #E53E3E; flex-shrink: 0; }
/* dashed track segments */
.wf-rtrack { flex: 1; border-top: 2px dashed #FCA5A5; }
/* center label */
.wf-rlbl { display: flex; align-items: center; gap: 5px; font-size: 11px; font-weight: 700; color: #C53030; white-space: nowrap; flex-shrink: 0; }
</style>
