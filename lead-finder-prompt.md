<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LMLFM — The Lean Mean Lead Finder Machine</title>
<link href="https://fonts.googleapis.com/css2?family=Share+Tech+Mono&family=Orbitron:wght@400;700;900&family=VT323&display=swap" rel="stylesheet">
<style>
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --black: #040506;
  --dark: #070a08;
  --panel: #0a0f0b;
  --panel2: #0d140e;
  --border: #1a2e1c;
  --border2: #243d26;
  --green: #00ff41;
  --green2: #00cc33;
  --green3: #39ff14;
  --green-dim: rgba(0,255,65,0.07);
  --green-glow: rgba(0,255,65,0.15);
  --amber: #ffb700;
  --red: #ff2442;
  --cyan: #00f5ff;
  --muted: #2a4a2d;
  --muted2: #3d6640;
  --text: #b8ffca;
  --text2: #7ab87e;
  --dim: #3a5c3e;
}

html { scroll-behavior: smooth; }

body {
  font-family: 'Share Tech Mono', monospace;
  background: var(--black);
  color: var(--green);
  min-height: 100vh;
  overflow-x: hidden;
  position: relative;
}

/* CRT scanlines */
body::before {
  content: '';
  position: fixed; inset: 0;
  background: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 2px,
    rgba(0,0,0,0.08) 2px,
    rgba(0,0,0,0.08) 4px
  );
  pointer-events: none;
  z-index: 9998;
}

/* Screen flicker */
body::after {
  content: '';
  position: fixed; inset: 0;
  background: radial-gradient(ellipse at center, transparent 60%, rgba(0,0,0,0.5) 100%);
  pointer-events: none;
  z-index: 9997;
  animation: vignette-pulse 8s ease infinite;
}

@keyframes vignette-pulse {
  0%,100% { opacity: 1; }
  50% { opacity: 0.85; }
}

/* Grid bg */
.bg-grid {
  position: fixed; inset: 0;
  background-image:
    linear-gradient(rgba(0,255,65,0.03) 1px, transparent 1px),
    linear-gradient(90deg, rgba(0,255,65,0.03) 1px, transparent 1px);
  background-size: 32px 32px;
  pointer-events: none;
  z-index: 0;
}

/* Glitch animation */
@keyframes glitch {
  0%,90%,100% { clip-path: none; transform: none; }
  91% { clip-path: polygon(0 20%, 100% 20%, 100% 40%, 0 40%); transform: translate(-3px, 0); }
  93% { clip-path: polygon(0 60%, 100% 60%, 100% 75%, 0 75%); transform: translate(3px, 0); }
  95% { clip-path: polygon(0 5%, 100% 5%, 100% 15%, 0 15%); transform: translate(-2px, 0); }
}

@keyframes blink-cursor {
  0%,49% { opacity: 1; }
  50%,100% { opacity: 0; }
}

@keyframes scan-line {
  0% { top: -10%; }
  100% { top: 110%; }
}

@keyframes data-stream {
  from { transform: translateY(-100%); }
  to { transform: translateY(100vh); }
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(16px); }
  to { opacity: 1; transform: translateY(0); }
}

@keyframes pulse-glow {
  0%,100% { box-shadow: 0 0 8px rgba(0,255,65,0.3); }
  50% { box-shadow: 0 0 24px rgba(0,255,65,0.7), 0 0 48px rgba(0,255,65,0.2); }
}

@keyframes rotate-radar {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes count-up {
  from { opacity: 0; transform: scale(0.8); }
  to { opacity: 1; transform: scale(1); }
}

/* ── HEADER ── */
.header {
  position: relative; z-index: 10;
  border-bottom: 1px solid var(--border);
  padding: 0 32px;
  background: var(--dark);
  display: flex;
  align-items: stretch;
  justify-content: space-between;
  overflow: hidden;
}

.header::after {
  content: '';
  position: absolute;
  bottom: 0; left: 0; right: 0;
  height: 1px;
  background: linear-gradient(90deg, transparent, var(--green), transparent);
  animation: scan-h 4s ease infinite;
}

@keyframes scan-h {
  0% { transform: scaleX(0); transform-origin: left; }
  50% { transform: scaleX(1); transform-origin: left; }
  51% { transform: scaleX(1); transform-origin: right; }
  100% { transform: scaleX(0); transform-origin: right; }
}

.header-brand {
  padding: 20px 0;
  border-right: 1px solid var(--border);
  padding-right: 32px;
  margin-right: 32px;
}

.brand-title {
  font-family: 'Orbitron', monospace;
  font-size: 11px;
  font-weight: 900;
  letter-spacing: 4px;
  color: var(--green);
  text-transform: uppercase;
  line-height: 1;
  animation: glitch 7s infinite;
}

.brand-sub {
  font-family: 'VT323', monospace;
  font-size: 22px;
  color: var(--green3);
  letter-spacing: 1px;
  margin-top: 2px;
  text-shadow: 0 0 12px rgba(57,255,20,0.6);
}

.header-status {
  display: flex;
  align-items: center;
  gap: 24px;
  flex: 1;
  padding: 16px 0;
}

.status-item {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.si-label {
  font-size: 9px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--muted2);
}

.si-value {
  font-family: 'Orbitron', monospace;
  font-size: 14px;
  font-weight: 700;
  color: var(--green);
}

.si-value.amber { color: var(--amber); }
.si-value.cyan { color: var(--cyan); }
.si-value.red { color: var(--red); }

.header-controls {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px 0;
}

/* System status dots */
.sys-dots {
  display: flex; gap: 6px; align-items: center;
}
.sys-dot {
  width: 8px; height: 8px; border-radius: 50%;
}
.sys-dot.green { background: var(--green); animation: pulse-glow 2s ease infinite; }
.sys-dot.amber { background: var(--amber); }
.sys-dot.red { background: var(--red); }

/* ── MAIN LAYOUT ── */
.app-body {
  position: relative; z-index: 1;
  display: grid;
  grid-template-columns: 340px 1fr 280px;
  grid-template-rows: auto 1fr;
  height: calc(100vh - 73px);
  gap: 0;
}

/* ── PANEL BASE ── */
.panel {
  border-right: 1px solid var(--border);
  background: var(--panel);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.panel:last-child { border-right: none; }

.panel-header {
  padding: 14px 18px;
  border-bottom: 1px solid var(--border);
  background: var(--dark);
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-shrink: 0;
}

.ph-title {
  font-family: 'Orbitron', monospace;
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 3px;
  text-transform: uppercase;
  color: var(--green);
  display: flex;
  align-items: center;
  gap: 8px;
}

.ph-title::before {
  content: '▶';
  font-size: 8px;
  color: var(--green2);
}

.ph-badge {
  font-size: 10px;
  padding: 2px 8px;
  border: 1px solid var(--border2);
  color: var(--text2);
  letter-spacing: 1px;
}

/* ── LEFT PANEL: TARGETING SYSTEM ── */
.targeting-panel { grid-row: 1 / 3; }

.panel-body { flex: 1; overflow-y: auto; padding: 16px; display: flex; flex-direction: column; gap: 14px; }
.panel-body::-webkit-scrollbar { width: 3px; }
.panel-body::-webkit-scrollbar-thumb { background: var(--border2); }

.field-group { display: flex; flex-direction: column; gap: 6px; }

.field-label {
  font-size: 9px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--muted2);
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.field-required { color: var(--green); }

.terminal-input {
  background: var(--dark);
  border: 1px solid var(--border2);
  border-radius: 0;
  color: var(--green);
  font-family: 'Share Tech Mono', monospace;
  font-size: 13px;
  padding: 9px 12px;
  width: 100%;
  outline: none;
  transition: border-color 0.15s, box-shadow 0.15s;
  appearance: none;
}

.terminal-input::placeholder { color: var(--muted); }
.terminal-input:focus { border-color: var(--green); box-shadow: 0 0 12px rgba(0,255,65,0.12); }

.terminal-select {
  background: var(--dark);
  border: 1px solid var(--border2);
  color: var(--green);
  font-family: 'Share Tech Mono', monospace;
  font-size: 13px;
  padding: 9px 12px;
  width: 100%;
  outline: none;
  cursor: pointer;
  transition: border-color 0.15s;
  -webkit-appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='10' height='6'%3E%3Cpath d='M0 0l5 6 5-6z' fill='%2300ff41'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 12px center;
}

.terminal-select:focus { border-color: var(--green); }
.terminal-select option { background: var(--dark); }

/* Tags input */
.tags-wrap {
  background: var(--dark);
  border: 1px solid var(--border2);
  padding: 6px 8px;
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  min-height: 42px;
  cursor: text;
  transition: border-color 0.15s;
}

.tags-wrap:focus-within { border-color: var(--green); }

.tag {
  display: flex;
  align-items: center;
  gap: 5px;
  background: var(--green-dim);
  border: 1px solid var(--border2);
  color: var(--green);
  font-size: 11px;
  padding: 2px 8px;
  cursor: default;
}

.tag-x {
  cursor: pointer;
  color: var(--muted2);
  font-size: 12px;
  line-height: 1;
}
.tag-x:hover { color: var(--red); }

.tags-input {
  background: transparent;
  border: none;
  outline: none;
  color: var(--green);
  font-family: 'Share Tech Mono', monospace;
  font-size: 12px;
  min-width: 80px;
  flex: 1;
}

/* Slider */
.range-wrap { position: relative; }
.terminal-range {
  width: 100%;
  -webkit-appearance: none;
  height: 3px;
  background: var(--border2);
  outline: none;
  cursor: pointer;
}
.terminal-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 14px; height: 14px;
  background: var(--green);
  border-radius: 0;
  cursor: pointer;
  box-shadow: 0 0 8px rgba(0,255,65,0.5);
}
.range-value {
  position: absolute;
  right: 0; top: -20px;
  font-size: 11px;
  color: var(--green);
  font-family: 'Orbitron', monospace;
}

/* Score filter chips */
.score-chips { display: flex; gap: 6px; flex-wrap: wrap; }
.score-chip {
  padding: 4px 10px;
  border: 1px solid var(--border2);
  background: transparent;
  color: var(--muted2);
  font-family: 'Share Tech Mono', monospace;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.15s;
  letter-spacing: 1px;
}
.score-chip:hover { border-color: var(--green); color: var(--green); }
.score-chip.active { background: var(--green-dim); border-color: var(--green); color: var(--green); }

/* LAUNCH BUTTON */
.launch-btn {
  width: 100%;
  padding: 16px;
  background: var(--green);
  color: var(--black);
  border: none;
  font-family: 'Orbitron', monospace;
  font-size: 13px;
  font-weight: 900;
  letter-spacing: 4px;
  text-transform: uppercase;
  cursor: pointer;
  transition: all 0.15s;
  position: relative;
  overflow: hidden;
  flex-shrink: 0;
  margin-top: auto;
}

.launch-btn::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(90deg, transparent 0%, rgba(255,255,255,0.2) 50%, transparent 100%);
  transform: translateX(-100%);
  transition: transform 0.5s;
}

.launch-btn:hover { background: var(--green3); box-shadow: 0 0 30px rgba(57,255,20,0.5); transform: translateY(-1px); }
.launch-btn:hover::before { transform: translateX(100%); }
.launch-btn:active { transform: translateY(0); }
.launch-btn:disabled { background: var(--muted); cursor: not-allowed; box-shadow: none; transform: none; }

.launch-btn .btn-icon { margin-right: 8px; }

/* ── CENTER PANEL: RESULTS ── */
.results-panel {
  grid-row: 1 / 3;
  border-right: 1px solid var(--border);
  display: flex;
  flex-direction: column;
}

/* Toolbar */
.results-toolbar {
  padding: 10px 16px;
  border-bottom: 1px solid var(--border);
  background: var(--dark);
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
}

.toolbar-btn {
  padding: 5px 12px;
  border: 1px solid var(--border2);
  background: transparent;
  color: var(--text2);
  font-family: 'Share Tech Mono', monospace;
  font-size: 11px;
  cursor: pointer;
  transition: all 0.15s;
  display: flex; align-items: center; gap: 5px;
  letter-spacing: 1px;
}
.toolbar-btn:hover { border-color: var(--green); color: var(--green); background: var(--green-dim); }
.toolbar-btn.active { border-color: var(--green); color: var(--green); background: var(--green-dim); }
.toolbar-sep { width: 1px; height: 20px; background: var(--border); margin: 0 4px; }

.results-search {
  flex: 1;
  background: var(--panel);
  border: 1px solid var(--border2);
  color: var(--green);
  font-family: 'Share Tech Mono', monospace;
  font-size: 12px;
  padding: 5px 12px;
  outline: none;
}
.results-search::placeholder { color: var(--muted); }
.results-search:focus { border-color: var(--green2); }

/* Results table */
.results-table-wrap { flex: 1; overflow-y: auto; }
.results-table-wrap::-webkit-scrollbar { width: 3px; }
.results-table-wrap::-webkit-scrollbar-thumb { background: var(--border2); }

table.leads-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
}

.leads-table thead {
  position: sticky; top: 0;
  background: var(--dark);
  z-index: 2;
}

.leads-table th {
  padding: 10px 14px;
  text-align: left;
  font-size: 9px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--muted2);
  border-bottom: 1px solid var(--border);
  font-weight: 400;
  white-space: nowrap;
  cursor: pointer;
  user-select: none;
  transition: color 0.15s;
}
.leads-table th:hover { color: var(--green); }
.leads-table th.sorted { color: var(--green); }

.leads-table td {
  padding: 11px 14px;
  border-bottom: 1px solid rgba(26,46,28,0.5);
  vertical-align: middle;
  white-space: nowrap;
}

.leads-table tr {
  cursor: pointer;
  transition: background 0.1s;
}
.leads-table tr:hover td { background: var(--green-dim); }
.leads-table tr.selected td { background: rgba(0,255,65,0.08); border-left: 2px solid var(--green); }

/* Lead name cell */
.lead-name-cell { display: flex; align-items: center; gap: 10px; }
.lead-avatar {
  width: 28px; height: 28px;
  background: var(--dark);
  border: 1px solid var(--border2);
  display: flex; align-items: center; justify-content: center;
  font-size: 10px;
  color: var(--green2);
  flex-shrink: 0;
  font-family: 'Orbitron', monospace;
}
.lead-name { color: var(--text); font-size: 12.5px; }
.lead-company { color: var(--dim); font-size: 10px; margin-top: 1px; }

/* Score bar */
.score-cell { display: flex; align-items: center; gap: 8px; }
.score-num {
  font-family: 'Orbitron', monospace;
  font-size: 11px;
  font-weight: 700;
  width: 28px;
  text-align: right;
}
.score-num.hot { color: var(--green3); }
.score-num.warm { color: var(--amber); }
.score-num.cold { color: var(--dim); }

.score-bar-bg {
  width: 50px; height: 4px;
  background: var(--border);
  position: relative;
  overflow: hidden;
}
.score-bar-fill {
  position: absolute;
  top: 0; left: 0; height: 100%;
  transition: width 0.5s ease;
}
.fill-hot { background: var(--green3); box-shadow: 0 0 6px rgba(57,255,20,0.5); }
.fill-warm { background: var(--amber); }
.fill-cold { background: var(--dim); }

/* Status badge */
.status-badge {
  display: inline-block;
  padding: 2px 8px;
  font-size: 9px;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  border: 1px solid;
}
.sb-new { border-color: var(--cyan); color: var(--cyan); }
.sb-contacted { border-color: var(--amber); color: var(--amber); }
.sb-qualified { border-color: var(--green); color: var(--green); }
.sb-cold { border-color: var(--dim); color: var(--dim); }

/* Intent signal */
.intent-dots { display: flex; gap: 3px; }
.intent-dot {
  width: 8px; height: 8px;
  border-radius: 50%;
  border: 1px solid var(--border2);
}
.intent-dot.lit { background: var(--green); box-shadow: 0 0 4px rgba(0,255,65,0.6); border-color: var(--green); }
.intent-dot.dim-lit { background: var(--amber); border-color: var(--amber); }

/* Empty state */
.empty-state {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 40px;
  text-align: center;
  color: var(--muted2);
}

.empty-radar {
  width: 120px; height: 120px;
  border: 2px solid var(--border2);
  border-radius: 50%;
  position: relative;
  margin: 0 auto 24px;
}

.empty-radar::before {
  content: '';
  position: absolute;
  inset: 16px;
  border: 1px solid var(--border2);
  border-radius: 50%;
}

.empty-radar::after {
  content: '';
  position: absolute;
  top: 50%; left: 50%;
  width: 50%; height: 2px;
  background: linear-gradient(90deg, transparent, var(--green));
  transform-origin: left center;
  animation: rotate-radar 2s linear infinite;
}

.radar-center {
  position: absolute;
  top: 50%; left: 50%;
  width: 6px; height: 6px;
  background: var(--green);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  box-shadow: 0 0 8px rgba(0,255,65,0.8);
}

.empty-title {
  font-family: 'Orbitron', monospace;
  font-size: 12px;
  letter-spacing: 3px;
  color: var(--text2);
  margin-bottom: 8px;
  text-transform: uppercase;
}

.empty-sub { font-size: 12px; color: var(--dim); line-height: 1.6; }

/* Results footer */
.results-footer {
  padding: 10px 16px;
  border-top: 1px solid var(--border);
  background: var(--dark);
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 10px;
  color: var(--muted2);
  flex-shrink: 0;
}

.results-footer .count { color: var(--green); font-family: 'Orbitron', monospace; }

/* ── RIGHT PANEL: INTELLIGENCE ── */
.intel-panel {
  grid-row: 1 / 3;
  background: var(--panel);
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.intel-body { flex: 1; overflow-y: auto; padding: 0; }
.intel-body::-webkit-scrollbar { width: 3px; }
.intel-body::-webkit-scrollbar-thumb { background: var(--border2); }

/* No selection state */
.intel-empty {
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  height: 100%; padding: 32px;
  text-align: center; color: var(--dim);
}
.intel-empty-icon { font-size: 32px; margin-bottom: 12px; opacity: 0.4; }
.intel-empty-text { font-size: 11px; line-height: 1.8; }

/* Lead detail */
.lead-detail { padding: 16px; }

.ld-name {
  font-family: 'VT323', monospace;
  font-size: 26px;
  color: var(--green3);
  text-shadow: 0 0 10px rgba(57,255,20,0.4);
  line-height: 1;
  margin-bottom: 3px;
}

.ld-title-company { font-size: 11px; color: var(--text2); margin-bottom: 16px; }

.ld-score-ring {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 12px;
  background: var(--dark);
  border: 1px solid var(--border2);
  margin-bottom: 14px;
}

.score-ring-val {
  font-family: 'Orbitron', monospace;
  font-size: 36px;
  font-weight: 900;
  line-height: 1;
}
.score-ring-label { font-size: 10px; color: var(--muted2); letter-spacing: 1px; text-transform: uppercase; }
.score-ring-sub { font-size: 11px; color: var(--text2); margin-top: 4px; }

.ld-section {
  margin-bottom: 14px;
  border: 1px solid var(--border);
  background: var(--dark);
}

.ld-section-title {
  padding: 7px 12px;
  border-bottom: 1px solid var(--border);
  font-size: 9px;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: var(--muted2);
  background: var(--panel2);
}

.ld-section-body { padding: 10px 12px; }

.ld-row {
  display: flex;
  justify-content: space-between;
  font-size: 11.5px;
  padding: 4px 0;
  border-bottom: 1px solid rgba(26,46,28,0.4);
}
.ld-row:last-child { border-bottom: none; }
.ld-key { color: var(--dim); }
.ld-val { color: var(--text); text-align: right; max-width: 55%; }

/* Signal bars */
.signals-grid { display: flex; flex-direction: column; gap: 8px; }
.signal-item { }
.signal-label { font-size: 10px; color: var(--muted2); margin-bottom: 4px; display: flex; justify-content: space-between; }
.signal-bar-bg { height: 5px; background: var(--border); position: relative; }
.signal-bar-fill { height: 100%; background: var(--green); transition: width 0.8s ease; }
.signal-bar-fill.amber { background: var(--amber); }
.signal-bar-fill.dim { background: var(--dim); }

/* AI summary */
.ai-summary {
  font-size: 12px;
  color: var(--text2);
  line-height: 1.7;
  padding: 10px 12px;
  border-left: 2px solid var(--green);
  background: var(--green-dim);
}

.ai-typing-cursor {
  display: inline-block;
  width: 8px; height: 12px;
  background: var(--green);
  animation: blink-cursor 0.8s infinite;
  vertical-align: middle;
  margin-left: 2px;
}

/* Action buttons */
.ld-actions { display: flex; flex-direction: column; gap: 8px; padding: 12px; border-top: 1px solid var(--border); flex-shrink: 0; }

.action-btn {
  width: 100%;
  padding: 10px;
  background: transparent;
  border: 1px solid var(--border2);
  color: var(--text2);
  font-family: 'Share Tech Mono', monospace;
  font-size: 11px;
  cursor: pointer;
  letter-spacing: 1px;
  text-transform: uppercase;
  transition: all 0.15s;
  display: flex; align-items: center; justify-content: center; gap: 8px;
}
.action-btn:hover { border-color: var(--green); color: var(--green); background: var(--green-dim); }
.action-btn.primary { background: var(--green-dim); border-color: var(--green2); color: var(--green); }
.action-btn.primary:hover { background: rgba(0,255,65,0.15); border-color: var(--green3); box-shadow: 0 0 12px rgba(0,255,65,0.2); }

/* ── TERMINAL LOG (bottom of center, above table) ── */
.terminal-log {
  height: 100px;
  border-top: 1px solid var(--border);
  background: var(--dark);
  padding: 8px 14px;
  overflow-y: auto;
  flex-shrink: 0;
  font-size: 11px;
  color: var(--green2);
  line-height: 1.7;
}
.terminal-log::-webkit-scrollbar { width: 3px; }
.terminal-log::-webkit-scrollbar-thumb { background: var(--border2); }

.log-line { display: flex; gap: 10px; }
.log-ts { color: var(--dim); flex-shrink: 0; }
.log-msg.success { color: var(--green); }
.log-msg.warn { color: var(--amber); }
.log-msg.error { color: var(--red); }
.log-msg.info { color: var(--cyan); }
.log-msg.dim { color: var(--dim); }

/* Loading overlay */
.loading-overlay {
  position: absolute; inset: 0;
  background: rgba(4,5,6,0.9);
  display: flex; flex-direction: column;
  align-items: center; justify-content: center;
  z-index: 50;
  display: none;
}

.loading-text {
  font-family: 'Orbitron', monospace;
  font-size: 13px;
  letter-spacing: 4px;
  color: var(--green);
  margin-bottom: 20px;
  animation: blink-cursor 1s infinite;
}

.loading-bar-bg {
  width: 300px; height: 4px;
  background: var(--border2);
  position: relative;
  overflow: hidden;
}

.loading-bar-fill {
  position: absolute; top: 0; left: -100%;
  width: 100%; height: 100%;
  background: linear-gradient(90deg, transparent, var(--green), transparent);
  animation: loading-sweep 1.5s ease infinite;
}

@keyframes loading-sweep {
  from { left: -100%; }
  to { left: 100%; }
}

.loading-steps { margin-top: 16px; font-size: 11px; color: var(--dim); text-align: center; line-height: 2; }

/* Notification toast */
.toast {
  position: fixed;
  bottom: 24px; right: 24px;
  background: var(--dark);
  border: 1px solid var(--green);
  color: var(--green);
  padding: 12px 20px;
  font-size: 12px;
  letter-spacing: 1px;
  z-index: 9999;
  box-shadow: 0 0 20px rgba(0,255,65,0.2);
  animation: toast-in 0.3s ease both;
  display: none;
}

@keyframes toast-in {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Checkbox */
.checkbox-wrap {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  font-size: 12px;
  color: var(--text2);
}
.terminal-checkbox {
  width: 14px; height: 14px;
  background: var(--dark);
  border: 1px solid var(--border2);
  appearance: none;
  cursor: pointer;
  position: relative;
  flex-shrink: 0;
}
.terminal-checkbox:checked { background: var(--green-dim); border-color: var(--green); }
.terminal-checkbox:checked::after {
  content: '✓';
  position: absolute;
  top: -2px; left: 1px;
  font-size: 12px;
  color: var(--green);
}

/* Divider */
.divider {
  height: 1px;
  background: var(--border);
  margin: 4px 0;
}
</style>
</head>
<body>
<div class="bg-grid"></div>

<!-- TOAST -->
<div class="toast" id="toast"></div>

<!-- ████ HEADER ████ -->
<header class="header">
  <div class="header-brand">
    <div class="brand-title">LMLFM // v2.4.1</div>
    <div class="brand-sub">⚡ LEAN MEAN LEAD FINDER</div>
  </div>

  <div class="header-status">
    <div class="status-item">
      <div class="si-label">System</div>
      <div class="si-value">ONLINE</div>
    </div>
    <div class="status-item">
      <div class="si-label">Leads Found</div>
      <div class="si-value" id="hdr-count">0</div>
    </div>
    <div class="status-item">
      <div class="si-label">Avg Score</div>
      <div class="si-value amber" id="hdr-score">—</div>
    </div>
    <div class="status-item">
      <div class="si-label">Hot Leads</div>
      <div class="si-value cyan" id="hdr-hot">0</div>
    </div>
    <div class="status-item">
      <div class="si-label">Last Scan</div>
      <div class="si-value dim" id="hdr-time" style="font-size:11px; color:var(--dim)">—</div>
    </div>
  </div>

  <div class="header-controls">
    <div class="sys-dots">
      <div class="sys-dot green"></div>
      <div class="sys-dot amber"></div>
      <div class="sys-dot red"></div>
    </div>
  </div>
</header>

<!-- ████ BODY ████ -->
<div class="app-body">

  <!-- ── LEFT: TARGETING ── -->
  <div class="panel targeting-panel">
    <div class="panel-header">
      <div class="ph-title">Target Parameters</div>
      <div class="ph-badge">CTRL</div>
    </div>

    <div class="panel-body" id="target-body">

      <div class="field-group">
        <div class="field-label">
          Industry / Niche <span class="field-required">*</span>
        </div>
        <input class="terminal-input" id="f-industry" type="text" placeholder="e.g. SaaS, Dental, Real Estate…" value="">
      </div>

      <div class="field-group">
        <div class="field-label">Target Location</div>
        <input class="terminal-input" id="f-location" type="text" placeholder="e.g. Melbourne, Australia">
      </div>

      <div class="field-group">
        <div class="field-label">Company Size</div>
        <select class="terminal-select" id="f-size">
          <option value="">Any Size</option>
          <option value="1-10">1–10 (Solo / Micro)</option>
          <option value="11-50">11–50 (Small)</option>
          <option value="51-200">51–200 (Mid-market)</option>
          <option value="201-1000">201–1000 (Growth)</option>
          <option value="1000+">1000+ (Enterprise)</option>
        </select>
      </div>

      <div class="field-group">
        <div class="field-label">Job Titles / Roles</div>
        <div class="tags-wrap" id="titles-tags" onclick="document.getElementById('titles-input').focus()">
          <input class="tags-input" id="titles-input" placeholder="Add title, press Enter" onkeydown="addTag(event,'titles')">
        </div>
      </div>

      <div class="field-group">
        <div class="field-label">Keywords / Pain Points</div>
        <div class="tags-wrap" id="kw-tags" onclick="document.getElementById('kw-input').focus()">
          <input class="tags-input" id="kw-input" placeholder="Add keyword, press Enter" onkeydown="addTag(event,'kw')">
        </div>
      </div>

      <div class="field-group">
        <div class="field-label">
          Min. Lead Score <span id="score-label" style="color:var(--green); font-family:'Orbitron',monospace; font-size:11px">60</span>
        </div>
        <div class="range-wrap">
          <input class="terminal-range" type="range" min="0" max="100" value="60" id="f-score" oninput="document.getElementById('score-label').textContent=this.value">
        </div>
      </div>

      <div class="field-group">
        <div class="field-label">Lead Temperature</div>
        <div class="score-chips" id="temp-chips">
          <button class="score-chip active" data-val="hot" onclick="toggleChip(this,'temp')">🔥 HOT</button>
          <button class="score-chip active" data-val="warm" onclick="toggleChip(this,'temp')">⚡ WARM</button>
          <button class="score-chip" data-val="cold" onclick="toggleChip(this,'temp')">❄ COLD</button>
        </div>
      </div>

      <div class="field-group">
        <div class="field-label">Number of Leads</div>
        <select class="terminal-select" id="f-count">
          <option value="5">5 leads</option>
          <option value="10" selected>10 leads</option>
          <option value="15">15 leads</option>
          <option value="20">20 leads</option>
        </select>
      </div>

      <div class="divider"></div>

      <div class="field-group">
        <div class="field-label">Options</div>
        <label class="checkbox-wrap">
          <input type="checkbox" class="terminal-checkbox" id="f-enrich" checked>
          Generate AI enrichment data
        </label>
        <label class="checkbox-wrap" style="margin-top:8px">
          <input type="checkbox" class="terminal-checkbox" id="f-signals" checked>
          Include buying signal analysis
        </label>
      </div>

    </div>

    <div style="padding:14px">
      <button class="launch-btn" id="launch-btn" onclick="runSearch()">
        <span class="btn-icon">⚡</span> INITIATE SCAN
      </button>
    </div>
  </div>

  <!-- ── CENTER: RESULTS ── -->
  <div class="results-panel" style="position:relative">

    <div class="panel-header">
      <div class="ph-title">Target Acquisition Feed</div>
      <div class="ph-badge" id="result-count">0 TARGETS</div>
    </div>

    <div class="results-toolbar">
      <button class="toolbar-btn active" onclick="filterTable('all',this)">ALL</button>
      <button class="toolbar-btn" onclick="filterTable('hot',this)">🔥 HOT</button>
      <button class="toolbar-btn" onclick="filterTable('warm',this)">⚡ WARM</button>
      <button class="toolbar-btn" onclick="filterTable('cold',this)">❄ COLD</button>
      <div class="toolbar-sep"></div>
      <input class="results-search" id="results-search" placeholder="⌕  Search targets…" oninput="searchTable(this.value)">
      <div class="toolbar-sep"></div>
      <button class="toolbar-btn" onclick="exportCSV()" title="Export CSV">⬇ CSV</button>
      <button class="toolbar-btn" onclick="clearResults()" title="Clear">✕ CLR</button>
    </div>

    <!-- Results table OR empty state -->
    <div id="table-area" style="flex:1; display:flex; flex-direction:column; overflow:hidden">
      <div class="empty-state" id="empty-state">
        <div class="empty-radar">
          <div class="radar-center"></div>
        </div>
        <div class="empty-title">Awaiting Target Lock</div>
        <div class="empty-sub">Configure targeting parameters<br>and initiate scan to find leads.</div>
      </div>
      <div class="results-table-wrap" id="results-table-wrap" style="display:none">
        <table class="leads-table" id="leads-table">
          <thead>
            <tr>
              <th onclick="sortTable('name')">Name / Company</th>
              <th onclick="sortTable('score')" class="sorted">Score ↓</th>
              <th onclick="sortTable('title')">Title</th>
              <th onclick="sortTable('status')">Status</th>
              <th onclick="sortTable('intent')">Intent</th>
              <th>Contact</th>
            </tr>
          </thead>
          <tbody id="leads-tbody"></tbody>
        </table>
      </div>
    </div>

    <!-- Terminal log -->
    <div class="terminal-log" id="terminal-log">
      <div class="log-line"><span class="log-ts">00:00:00</span><span class="log-msg dim">// LMLFM System initialized. Awaiting scan parameters.</span></div>
    </div>

    <div class="results-footer">
      <div>LMLFM v2.4.1 // AI-POWERED LEAD INTEL</div>
      <div><span class="count" id="footer-count">0</span> TARGETS ACQUIRED</div>
    </div>

    <!-- Loading overlay -->
    <div class="loading-overlay" id="loading-overlay">
      <div class="loading-text" id="loading-text">SCANNING...</div>
      <div class="loading-bar-bg"><div class="loading-bar-fill"></div></div>
      <div class="loading-steps" id="loading-steps"></div>
    </div>
  </div>

  <!-- ── RIGHT: INTELLIGENCE ── -->
  <div class="intel-panel">
    <div class="panel-header">
      <div class="ph-title">Lead Intelligence</div>
      <div class="ph-badge">INTEL</div>
    </div>

    <div class="intel-body" id="intel-body">
      <div class="intel-empty">
        <div class="intel-empty-icon">◎</div>
        <div class="intel-empty-text">Select a target to view<br>full intelligence report</div>
      </div>
    </div>

    <div class="ld-actions" id="ld-actions" style="display:none">
      <button class="action-btn primary" onclick="copyOutreach()">⚡ COPY AI OUTREACH</button>
      <button class="action-btn" onclick="addToList()">+ ADD TO HIT LIST</button>
      <button class="action-btn" onclick="markContacted()">✓ MARK CONTACTED</button>
    </div>
  </div>

</div>

<script>
// ── STATE ──
let allLeads = [];
let filteredLeads = [];
let selectedLead = null;
let currentFilter = 'all';
let sortCol = 'score';
let sortAsc = false;
let tags = { titles: [], kw: [] };
let hitList = [];

// ── TAG INPUT ──
function addTag(e, group) {
  if (e.key !== 'Enter') return;
  const input = document.getElementById(group === 'titles' ? 'titles-input' : 'kw-input');
  const val = input.value.trim();
  if (!val) return;
  tags[group].push(val);
  input.value = '';
  renderTags(group);
}

function removeTag(group, idx) {
  tags[group].splice(idx, 1);
  renderTags(group);
}

function renderTags(group) {
  const wrap = document.getElementById(group === 'titles' ? 'titles-tags' : 'kw-tags');
  const input = document.getElementById(group === 'titles' ? 'titles-input' : 'kw-input');
  const tagEls = tags[group].map((t, i) => `
    <div class="tag">${t}<span class="tag-x" onclick="removeTag('${group}',${i})">×</span></div>
  `).join('');
  wrap.innerHTML = tagEls;
  wrap.appendChild(input);
}

// ── CHIPS ──
function toggleChip(el, group) {
  el.classList.toggle('active');
}

// ── LOGGING ──
function log(msg, type = 'dim') {
  const log = document.getElementById('terminal-log');
  const ts = new Date().toLocaleTimeString('en-AU', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
  const line = document.createElement('div');
  line.className = 'log-line';
  line.innerHTML = `<span class="log-ts">${ts}</span><span class="log-msg ${type}">${msg}</span>`;
  log.appendChild(line);
  log.scrollTop = log.scrollHeight;
}

// ── TOAST ──
function showToast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.style.display = 'block';
  setTimeout(() => { t.style.display = 'none'; }, 2500);
}

// ── MAIN SEARCH ──
async function runSearch() {
  const industry = document.getElementById('f-industry').value.trim();
  if (!industry) {
    showToast('⚠ Industry/Niche required');
    document.getElementById('f-industry').focus();
    return;
  }

  const location = document.getElementById('f-location').value.trim();
  const size = document.getElementById('f-size').value;
  const minScore = document.getElementById('f-score').value;
  const count = document.getElementById('f-count').value;
  const enrich = document.getElementById('f-enrich').checked;
  const signals = document.getElementById('f-signals').checked;
  const activeTemps = [...document.querySelectorAll('#temp-chips .score-chip.active')].map(c => c.dataset.val);
  const titlesList = tags.titles.join(', ') || 'Any decision maker';
  const kwList = tags.kw.join(', ') || 'None specified';

  // Show loading
  const overlay = document.getElementById('loading-overlay');
  overlay.style.display = 'flex';
  document.getElementById('launch-btn').disabled = true;

  const steps = [
    '▶ Initializing AI lead scanner…',
    '▶ Mapping industry landscape…',
    '▶ Identifying decision makers…',
    '▶ Scoring leads with intent signals…',
    '▶ Enriching company profiles…',
    '▶ Generating outreach intelligence…',
  ];

  let si = 0;
  const stepEl = document.getElementById('loading-steps');
  const stepInterval = setInterval(() => {
    if (si < steps.length) {
      stepEl.innerHTML += steps[si] + '\n';
      document.getElementById('loading-text').textContent = `SCANNING… ${Math.round((si/steps.length)*100)}%`;
      si++;
    }
  }, 700);

  log(`Scan initiated — Industry: ${industry} | Location: ${location || 'Global'} | Size: ${size || 'Any'} | Count: ${count}`, 'info');
  log(`Titles: ${titlesList} | Keywords: ${kwList}`, 'dim');

  const prompt = buildPrompt(industry, location, size, minScore, count, titlesList, kwList, activeTemps, enrich, signals);

  try {
    const res = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        model: 'claude-sonnet-4-20250514',
        max_tokens: 4000,
        messages: [{ role: 'user', content: prompt }]
      })
    });

    const data = await res.json();
    const raw = data.content?.map(b => b.text || '').join('') || '';

    clearInterval(stepInterval);

    // Extract JSON
    const jsonMatch = raw.match(/```json([\s\S]*?)```/) || raw.match(/\[[\s\S]*\]/);
    let leads = [];

    if (jsonMatch) {
      try {
        const jsonStr = jsonMatch[1] ? jsonMatch[1].trim() : jsonMatch[0];
        leads = JSON.parse(jsonStr);
      } catch(e) {
        // Try to find array directly
        const arrMatch = raw.match(/\[[\s\S]*\]/);
        if (arrMatch) leads = JSON.parse(arrMatch[0]);
      }
    }

    overlay.style.display = 'none';
    document.getElementById('launch-btn').disabled = false;

    if (leads.length > 0) {
      allLeads = leads;
      filteredLeads = [...leads];
      renderTable(filteredLeads);
      updateStats();
      log(`✓ Acquired ${leads.length} targets. Avg score: ${Math.round(leads.reduce((a,l)=>a+l.score,0)/leads.length)}`, 'success');
      log(`Hot leads: ${leads.filter(l=>l.temperature==='hot').length} | Warm: ${leads.filter(l=>l.temperature==='warm').length}`, 'info');
      showToast(`⚡ ${leads.length} targets acquired`);
    } else {
      log('⚠ No leads parsed from AI response. Try different parameters.', 'warn');
      showToast('⚠ Parse error — try again');
    }

  } catch(err) {
    clearInterval(stepInterval);
    overlay.style.display = 'none';
    document.getElementById('launch-btn').disabled = false;
    log(`✕ Scan failed: ${err.message}`, 'error');
    showToast('✕ Scan failed — check connection');
  }
}

function buildPrompt(industry, location, size, minScore, count, titles, keywords, temps, enrich, signals) {
  return `You are an elite B2B lead intelligence system. Generate ${count} realistic, detailed fictional lead profiles for the following criteria. These are fictional but HIGHLY realistic and useful for demonstrating a lead finder tool.

TARGETING CRITERIA:
- Industry/Niche: ${industry}
- Location: ${location || 'Global (mix of countries)'}
- Company Size: ${size || 'Mixed sizes'}
- Target Roles/Titles: ${titles}
- Keywords/Pain Points: ${keywords}
- Min Lead Score: ${minScore}
- Temperature Filter: ${temps.join(', ')}
- Include Enrichment: ${enrich}
- Include Buying Signals: ${signals}

Generate EXACTLY ${count} leads. Return ONLY a JSON array (no markdown, no explanation) with this exact structure for each lead:

[
  {
    "id": "lead_001",
    "firstName": "Sarah",
    "lastName": "Mitchell",
    "title": "Head of Marketing",
    "company": "Acme SaaS Co",
    "industry": "${industry}",
    "location": "Melbourne, Australia",
    "companySize": "51-200",
    "revenue": "$5M-$20M",
    "score": 87,
    "temperature": "hot",
    "status": "new",
    "email": "s.mitchell@acmesaas.com",
    "linkedin": "linkedin.com/in/sarahmitchell",
    "phone": "+61 4XX XXX XXX",
    "website": "acmesaas.com",
    "techStack": ["HubSpot", "Salesforce", "Slack"],
    "painPoints": ["Manual reporting", "Lead qualification time"],
    "buyingSignals": ["Recently hired 3 SDRs", "Posted VP Sales job listing", "Attended SaaStr 2024"],
    "intentScore": 82,
    "engagementScore": 74,
    "fitScore": 91,
    "lastActivity": "Visited pricing page 3x this week",
    "aiSummary": "Sarah leads a fast-growing SaaS team actively expanding their sales infrastructure. Recent hiring signals strong growth phase — ideal timing for outreach around sales automation and pipeline tooling.",
    "outreachAngle": "Reference her recent SDR hiring and offer to help them scale without burning headcount on manual tasks.",
    "tags": ["decision-maker", "growth-stage", "tech-forward"]
  }
]

Make all ${count} leads realistic and diverse. Vary scores, temperatures, companies, locations. Ensure at least ${Math.ceil(count*0.3)} are hot leads with score 75+. Return ONLY the JSON array.`;
}

// ── RENDER TABLE ──
function renderTable(leads) {
  const empty = document.getElementById('empty-state');
  const wrap = document.getElementById('results-table-wrap');

  if (leads.length === 0) {
    empty.style.display = 'flex';
    wrap.style.display = 'none';
    return;
  }

  empty.style.display = 'none';
  wrap.style.display = 'block';

  const tbody = document.getElementById('leads-tbody');
  tbody.innerHTML = leads.map((l, i) => {
    const temp = l.temperature || 'warm';
    const scoreClass = l.score >= 75 ? 'hot' : l.score >= 50 ? 'warm' : 'cold';
    const fillClass = `fill-${scoreClass}`;
    const intentDots = [1,2,3,4,5].map(d => {
      const lit = d <= Math.round((l.intentScore||60)/20);
      return `<div class="intent-dot ${lit ? (l.score>=75?'lit':'dim-lit') : ''}"></div>`;
    }).join('');
    const statusBadge = {new:'sb-new',contacted:'sb-contacted',qualified:'sb-qualified',cold:'sb-cold'}[l.status] || 'sb-new';
    const initials = (l.firstName?.[0]||'') + (l.lastName?.[0]||'?');

    return `<tr onclick="selectLead(${i})" id="row-${i}">
      <td>
        <div class="lead-name-cell">
          <div class="lead-avatar">${initials}</div>
          <div>
            <div class="lead-name">${l.firstName} ${l.lastName}</div>
            <div class="lead-company">${l.company}</div>
          </div>
        </div>
      </td>
      <td>
        <div class="score-cell">
          <div class="score-num ${scoreClass}">${l.score}</div>
          <div class="score-bar-bg"><div class="score-bar-fill ${fillClass}" style="width:${l.score}%"></div></div>
        </div>
      </td>
      <td style="color:var(--text2); font-size:11px; max-width:120px; overflow:hidden; text-overflow:ellipsis">${l.title}</td>
      <td><span class="status-badge ${statusBadge}">${l.status||'new'}</span></td>
      <td><div class="intent-dots">${intentDots}</div></td>
      <td style="color:var(--dim); font-size:10px">${l.email || '—'}</td>
    </tr>`;
  }).join('');

  document.getElementById('result-count').textContent = `${leads.length} TARGETS`;
  document.getElementById('footer-count').textContent = leads.length;
}

// ── SELECT LEAD ──
function selectLead(idx) {
  // Deselect old
  document.querySelectorAll('.leads-table tr.selected').forEach(r => r.classList.remove('selected'));
  const row = document.getElementById(`row-${idx}`);
  if (row) row.classList.add('selected');

  selectedLead = filteredLeads[idx];
  renderIntel(selectedLead);
  document.getElementById('ld-actions').style.display = 'flex';
  log(`Target selected: ${selectedLead.firstName} ${selectedLead.lastName} @ ${selectedLead.company}`, 'info');
}

// ── RENDER INTEL ──
function renderIntel(lead) {
  const body = document.getElementById('intel-body');
  const scoreClass = lead.score >= 75 ? 'hot' : lead.score >= 50 ? 'warm' : 'cold';
  const scoreColor = lead.score >= 75 ? 'var(--green3)' : lead.score >= 50 ? 'var(--amber)' : 'var(--dim)';

  const signals = lead.buyingSignals || [];
  const signalItems = signals.slice(0,4).map(s => `
    <div style="font-size:11px; color:var(--text2); padding:3px 0; border-bottom:1px solid rgba(26,46,28,0.4); display:flex; gap:6px; align-items:flex-start">
      <span style="color:var(--green); flex-shrink:0">▸</span> ${s}
    </div>
  `).join('');

  const techItems = (lead.techStack||[]).map(t =>
    `<span style="display:inline-block; border:1px solid var(--border2); padding:2px 7px; font-size:10px; color:var(--cyan); margin:2px 2px 2px 0">${t}</span>`
  ).join('');

  const painItems = (lead.painPoints||[]).map(p =>
    `<div style="font-size:11px; color:var(--amber); padding:3px 0; display:flex; gap:6px"><span>⚡</span>${p}</div>`
  ).join('');

  body.innerHTML = `
    <div class="lead-detail">
      <div class="ld-name">${lead.firstName} ${lead.lastName}</div>
      <div class="ld-title-company">${lead.title} @ ${lead.company} · ${lead.location}</div>

      <div class="ld-score-ring">
        <div>
          <div class="score-ring-val" style="color:${scoreColor}">${lead.score}</div>
          <div class="score-ring-label">Lead Score</div>
        </div>
        <div style="flex:1">
          <div class="score-ring-sub" style="color:${scoreColor}">${lead.score>=75?'🔥 HOT TARGET':lead.score>=50?'⚡ WARM LEAD':'❄ COLD LEAD'}</div>
          <div class="score-ring-sub">Intent: ${lead.intentScore||'—'} | Fit: ${lead.fitScore||'—'}</div>
          <div class="score-ring-sub" style="font-size:10px; color:var(--dim); margin-top:4px">${lead.lastActivity||''}</div>
        </div>
      </div>

      <div class="ld-section">
        <div class="ld-section-title">Company Profile</div>
        <div class="ld-section-body">
          <div class="ld-row"><span class="ld-key">Company</span><span class="ld-val">${lead.company}</span></div>
          <div class="ld-row"><span class="ld-key">Industry</span><span class="ld-val">${lead.industry||'—'}</span></div>
          <div class="ld-row"><span class="ld-key">Size</span><span class="ld-val">${lead.companySize||'—'}</span></div>
          <div class="ld-row"><span class="ld-key">Revenue</span><span class="ld-val">${lead.revenue||'—'}</span></div>
          <div class="ld-row"><span class="ld-key">Website</span><span class="ld-val" style="color:var(--cyan)">${lead.website||'—'}</span></div>
        </div>
      </div>

      <div class="ld-section">
        <div class="ld-section-title">Contact Info</div>
        <div class="ld-section-body">
          <div class="ld-row"><span class="ld-key">Email</span><span class="ld-val" style="color:var(--green)">${lead.email||'—'}</span></div>
          <div class="ld-row"><span class="ld-key">Phone</span><span class="ld-val">${lead.phone||'—'}</span></div>
          <div class="ld-row"><span class="ld-key">LinkedIn</span><span class="ld-val" style="color:var(--cyan); font-size:10px">${lead.linkedin||'—'}</span></div>
        </div>
      </div>

      ${signals.length ? `
      <div class="ld-section">
        <div class="ld-section-title">🔥 Buying Signals</div>
        <div class="ld-section-body">${signalItems}</div>
      </div>` : ''}

      ${lead.techStack?.length ? `
      <div class="ld-section">
        <div class="ld-section-title">Tech Stack</div>
        <div class="ld-section-body">${techItems}</div>
      </div>` : ''}

      ${lead.painPoints?.length ? `
      <div class="ld-section">
        <div class="ld-section-title">⚡ Pain Points</div>
        <div class="ld-section-body">${painItems}</div>
      </div>` : ''}

      <div class="ld-section">
        <div class="ld-section-title">Signal Breakdown</div>
        <div class="ld-section-body">
          <div class="signals-grid">
            <div class="signal-item">
              <div class="signal-label"><span>Intent Score</span><span style="color:var(--green)">${lead.intentScore||60}</span></div>
              <div class="signal-bar-bg"><div class="signal-bar-fill" style="width:${lead.intentScore||60}%"></div></div>
            </div>
            <div class="signal-item">
              <div class="signal-label"><span>Engagement</span><span style="color:var(--amber)">${lead.engagementScore||55}</span></div>
              <div class="signal-bar-bg"><div class="signal-bar-fill amber" style="width:${lead.engagementScore||55}%"></div></div>
            </div>
            <div class="signal-item">
              <div class="signal-label"><span>Profile Fit</span><span style="color:var(--green)">${lead.fitScore||70}</span></div>
              <div class="signal-bar-bg"><div class="signal-bar-fill" style="width:${lead.fitScore||70}%"></div></div>
            </div>
          </div>
        </div>
      </div>

      ${lead.aiSummary ? `
      <div class="ld-section">
        <div class="ld-section-title">AI Intelligence Summary</div>
        <div class="ld-section-body">
          <div class="ai-summary">${lead.aiSummary}</div>
        </div>
      </div>` : ''}

      ${lead.outreachAngle ? `
      <div class="ld-section">
        <div class="ld-section-title">⚡ Outreach Angle</div>
        <div class="ld-section-body">
          <div style="font-size:12px; color:var(--green3); line-height:1.7; padding:8px; background:var(--green-dim); border-left:2px solid var(--green3)">${lead.outreachAngle}</div>
        </div>
      </div>` : ''}

      ${lead.tags?.length ? `
      <div style="margin-top:12px; display:flex; gap:6px; flex-wrap:wrap">
        ${lead.tags.map(t=>`<span style="border:1px solid var(--border2); padding:2px 8px; font-size:10px; color:var(--dim)">#${t}</span>`).join('')}
      </div>` : ''}
    </div>
  `;
}

// ── FILTER ──
function filterTable(type, btn) {
  document.querySelectorAll('.toolbar-btn').forEach(b => b.classList.remove('active'));
  btn.classList.add('active');
  currentFilter = type;
  applyFilters();
}

function searchTable(q) {
  applyFilters(q);
}

function applyFilters(searchQ) {
  const q = searchQ !== undefined ? searchQ : document.getElementById('results-search').value;
  filteredLeads = allLeads.filter(l => {
    const matchFilter = currentFilter === 'all' || l.temperature === currentFilter;
    const matchSearch = !q || `${l.firstName} ${l.lastName} ${l.company} ${l.title}`.toLowerCase().includes(q.toLowerCase());
    return matchFilter && matchSearch;
  });
  renderTable(filteredLeads);
}

// ── SORT ──
function sortTable(col) {
  if (sortCol === col) sortAsc = !sortAsc;
  else { sortCol = col; sortAsc = false; }
  filteredLeads.sort((a, b) => {
    let av = a[col], bv = b[col];
    if (typeof av === 'string') av = av.toLowerCase();
    if (typeof bv === 'string') bv = bv.toLowerCase();
    return sortAsc ? (av > bv ? 1 : -1) : (av < bv ? 1 : -1);
  });
  renderTable(filteredLeads);
}

// ── UPDATE STATS ──
function updateStats() {
  const total = allLeads.length;
  const avg = total ? Math.round(allLeads.reduce((a,l)=>a+l.score,0)/total) : 0;
  const hot = allLeads.filter(l=>l.temperature==='hot').length;
  document.getElementById('hdr-count').textContent = total;
  document.getElementById('hdr-score').textContent = avg || '—';
  document.getElementById('hdr-hot').textContent = hot;
  document.getElementById('hdr-time').textContent = new Date().toLocaleTimeString('en-AU',{hour:'2-digit',minute:'2-digit'});
  document.getElementById('footer-count').textContent = total;
}

// ── ACTIONS ──
function copyOutreach() {
  if (!selectedLead) return;
  const msg = `Hi ${selectedLead.firstName},\n\nI noticed ${selectedLead.company} has been ${selectedLead.outreachAngle || 'growing rapidly'}.\n\n${selectedLead.aiSummary || ''}\n\nWould love to connect for a quick 15-min chat?\n\n[Your Name]`;
  navigator.clipboard.writeText(msg).then(() => showToast('⚡ Outreach copied to clipboard'));
}

function addToList() {
  if (!selectedLead || hitList.find(l=>l.id===selectedLead.id)) { showToast('⚠ Already in hit list'); return; }
  hitList.push(selectedLead);
  log(`+ ${selectedLead.firstName} ${selectedLead.lastName} added to hit list`, 'success');
  showToast(`+ Added to hit list (${hitList.length} total)`);
}

function markContacted() {
  if (!selectedLead) return;
  selectedLead.status = 'contacted';
  const idx = filteredLeads.indexOf(selectedLead);
  renderTable(filteredLeads);
  if (idx >= 0) selectLead(idx);
  log(`✓ ${selectedLead.firstName} ${selectedLead.lastName} marked as contacted`, 'success');
  showToast('✓ Marked as contacted');
}

function clearResults() {
  allLeads = []; filteredLeads = []; selectedLead = null;
  document.getElementById('empty-state').style.display = 'flex';
  document.getElementById('results-table-wrap').style.display = 'none';
  document.getElementById('intel-body').innerHTML = '<div class="intel-empty"><div class="intel-empty-icon">◎</div><div class="intel-empty-text">Select a target to view<br>full intelligence report</div></div>';
  document.getElementById('ld-actions').style.display = 'none';
  document.getElementById('result-count').textContent = '0 TARGETS';
  document.getElementById('footer-count').textContent = '0';
  document.getElementById('hdr-count').textContent = '0';
  document.getElementById('hdr-score').textContent = '—';
  document.getElementById('hdr-hot').textContent = '0';
  log('// Results cleared. Ready for new scan.', 'dim');
}

function exportCSV() {
  if (!allLeads.length) { showToast('⚠ No leads to export'); return; }
  const headers = ['Name','Title','Company','Industry','Location','Size','Revenue','Score','Temperature','Status','Email','Phone','LinkedIn','Website'];
  const rows = allLeads.map(l => [
    `${l.firstName} ${l.lastName}`, l.title, l.company, l.industry, l.location,
    l.companySize, l.revenue, l.score, l.temperature, l.status,
    l.email, l.phone, l.linkedin, l.website
  ].map(v => `"${(v||'').toString().replace(/"/g,'""')}"`).join(','));
  const csv = [headers.join(','), ...rows].join('\n');
  const blob = new Blob([csv], {type:'text/csv'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `LMLFM_leads_${Date.now()}.csv`;
  a.click();
  log(`↓ Exported ${allLeads.length} leads to CSV`, 'success');
  showToast(`↓ Exported ${allLeads.length} leads`);
}

// ── INIT LOG ──
setTimeout(() => log('⚡ All systems operational. Configure targeting parameters to begin.', 'success'), 500);
setTimeout(() => log('// Tip: Add job titles and keywords for higher quality leads', 'dim'), 1200);
</script>
</body>
</html>
