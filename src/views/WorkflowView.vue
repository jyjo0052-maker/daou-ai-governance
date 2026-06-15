<template>
  <div class="wf-page">

    <div class="wf-hero">
      <div>
        <div class="wf-eyebrow">DAOU Technology · AI Governance</div>
        <h2 class="wf-title">전체 서비스 워크플로우</h2>
        <p class="wf-sub">AI 기본법(2026.01.22 시행) 기반 다우기술 AI 거버넌스 운영 체계</p>
      </div>
    </div>

    <section class="wf-card">
      <div class="wf-diagram">

        <!-- Row 1: nodes + connectors (7 cells) -->

        <div class="wf-node" style="border-top-color:#1B6B3A">
          <div class="wf-node-icon" style="background:#1B6B3A">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="2" y="3" width="20" height="14" rx="2"/><path d="M8 21h8M12 17v4"/></svg>
          </div>
          <div class="wf-node-name">AI 운영부서</div>
          <div class="wf-node-desc">신규 AI 서비스 기획 및<br>등록 신청</div>
        </div>

        <div class="wf-conn">
          <div class="wf-conn-lbl wf-fwd">신규 등록 신청</div>
          <div class="wf-line"><div class="wf-track"></div><div class="wf-head"></div></div>
        </div>

        <div class="wf-node" style="border-top-color:#5B21B6">
          <div class="wf-node-icon" style="background:#5B21B6">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2 2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5"/><path d="M2 12l10 5 10-5"/></svg>
          </div>
          <div class="wf-node-name">법무팀</div>
          <div class="wf-node-desc">고영향 AI 판정 · 법률 검토<br>규제 항목 관리</div>
        </div>

        <div class="wf-conn">
          <div class="wf-conn-lbl wf-fwd">검토 요청</div>
          <div class="wf-line"><div class="wf-track"></div><div class="wf-head"></div></div>
        </div>

        <div class="wf-node wf-node-ours" style="border-top-color:#0D47A1">
          <div class="wf-node-icon" style="background:#0D47A1">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
          </div>
          <div class="wf-node-name">AX 기획팀</div>
          <div class="wf-node-desc">최종 승인 및<br>상시 규제 모니터링</div>
        </div>

        <div class="wf-conn wf-conn-bi">
          <div class="wf-conn-lbl wf-fwd">정기 보고</div>
          <div class="wf-line"><div class="wf-track"></div><div class="wf-head"></div></div>
          <div class="wf-conn-lbl wf-back">전략 방향</div>
          <div class="wf-line"><div class="wf-head-back"></div><div class="wf-track"></div></div>
        </div>

        <div class="wf-node" style="border-top-color:#263238">
          <div class="wf-node-icon" style="background:#263238">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
          </div>
          <div class="wf-node-name">경영진</div>
          <div class="wf-node-desc">현황 파악 및<br>전략 방향 의사결정</div>
        </div>

        <!--
          Row 2: 승인/반려 복귀 화살표
          AX기획팀 bottom-center → 아래로 꺾여 → AI 운영부서 bottom-center ↑
          Subgrid: 5 children fill cols 1–5 with exact same widths as row 1
        -->
        <div class="wf-return">
          <!-- col 1: AI 운영부서 — right half has upward arrowhead + border going right -->
          <div class="ret-c1">
            <div class="ret-c1-inner"></div>
          </div>
          <!-- cols 2–4: horizontal line with label -->
          <div class="ret-mid">
            <span class="ret-lbl">승인 / 반려 통보</span>
          </div>
          <!-- col 5: AX 기획팀 — left half has vertical drop + border going left -->
          <div class="ret-c5">
            <div class="ret-c5-inner"></div>
          </div>
        </div>

        <!-- cols 6–7 row 2: empty -->
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
.wf-title   { font-size: 24px; font-weight: 800; color: white; margin: 0 0 6px; }
.wf-sub     { font-size: 12px; color: #78909C; margin: 0; }

.wf-card { background: white; border: 1px solid var(--gray-200); border-radius: 12px; padding: 36px 28px 40px; }

/* 7-column grid, 2 rows */
.wf-diagram {
  display: grid;
  grid-template-columns: 1fr 80px 1fr 80px 1fr 96px 1fr;
  grid-template-rows: auto auto;
  align-items: center;
  overflow: visible;
}
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

/* Connectors */
.wf-conn { display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 5px; padding: 0 4px; }
.wf-conn-lbl { display: flex; align-items: center; gap: 3px; font-size: 9px; font-weight: 600; white-space: nowrap; }
.wf-fwd  { color: #1E40AF; }
.wf-back { color: var(--gray-500); }
.wf-line { display: flex; align-items: center; width: 100%; }
.wf-track { flex: 1; height: 2px; background: #BFDBFE; }
.wf-head      { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-left: 8px solid #93C5FD; }
.wf-head-back { width: 0; height: 0; border-top: 5px solid transparent; border-bottom: 5px solid transparent; border-right: 8px solid #93C5FD; }

/* ── Return arc (row 2, col 1–5) ──
   Uses CSS subgrid so each child inherits exact column widths from the parent grid.
   Visual path: AX기획팀 center-bottom → down → left → AI 운영부서 center-bottom ↑
*/
.wf-return {
  grid-column: 1 / 6;
  display: grid;
  grid-template-columns: subgrid;
  height: 44px;
  overflow: visible;
  margin-top: 4px;
}

/* col 1 (AI 운영부서): right half = vertical line + upward arrowhead */
.ret-c1 {
  grid-column: 1;
  display: flex;
  justify-content: flex-end; /* push inner to right half */
  align-items: stretch;
  overflow: visible;
}
.ret-c1-inner {
  width: 50%;
  border-left: 2px solid #BFDBFE;
  border-bottom: 2px solid #BFDBFE;
  position: relative;
  overflow: visible;
}
/* Upward arrowhead ↑ at the top of the vertical line (AI 운영부서 bottom center) */
.ret-c1-inner::before {
  content: '';
  position: absolute;
  top: -8px;
  left: -6px;
  width: 0; height: 0;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-bottom: 9px solid #93C5FD;
}

/* cols 2–4 (connector + 법무팀 + connector): horizontal line */
.ret-mid {
  grid-column: 2 / 5;
  border-bottom: 2px solid #BFDBFE;
  display: flex;
  align-items: flex-end;
  justify-content: center;
  padding-bottom: 7px;
}
.ret-lbl {
  font-size: 9px; font-weight: 600; color: #1E40AF; white-space: nowrap;
}

/* col 5 (AX 기획팀): left half = vertical drop + corner going left */
.ret-c5 {
  grid-column: 5;
  display: flex;
  justify-content: flex-start; /* push inner to left half */
  align-items: stretch;
}
.ret-c5-inner {
  width: 50%;
  border-right: 2px solid #BFDBFE;
  border-bottom: 2px solid #BFDBFE;
  border-radius: 0 0 8px 0;
}
</style>
