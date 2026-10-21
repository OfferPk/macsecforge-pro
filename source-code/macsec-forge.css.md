:root {  
  \--bg-primary: \#060913;  
  \--bg-card: \#0c1222;  
  \--bg-card-hover: \#111a33;  
  \--border-color: \#1d2847;  
  \--text-main: \#f1f5f9;  
  \--text-muted: \#94a3b8;  
  \--accent-emerald: \#10b981;  
  \--accent-cyan: \#06b6d4;  
  \--accent-purple: \#8b5cf6;  
  \--accent-amber: \#f59e0b;  
  \--accent-blue: \#3b82f6;  
  \--success: \#10b981;  
  \--warning: \#f59e0b;  
  \--danger: \#ef4444;  
  \--font-sans: \-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;  
  \--font-mono: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Liberation Mono", "Courier New", monospace;  
}

\* {  
  box-sizing: border-box;  
  margin: 0;  
  padding: 0;  
}

body {  
  background-color: var(--bg-primary);  
  color: var(--text-main);  
  font-family: var(--font-sans);  
  line-height: 1.5;  
  padding: 24px;  
}

.studio-container {  
  max-width: 1440px;  
  margin: 0 auto;  
}

/\* Header \*/  
.studio-header {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  padding-bottom: 20px;  
  border-bottom: 1px solid var(--border-color);  
  margin-bottom: 24px;  
}

.brand-area {  
  display: flex;  
  align-items: center;  
  gap: 14px;  
}

.brand-logo {  
  width: 44px;  
  height: 44px;  
  background: linear-gradient(135deg, var(--accent-emerald), var(--accent-cyan));  
  border-radius: 10px;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-weight: 900;  
  font-size: 22px;  
  color: \#000;  
  box-shadow: 0 0 18px rgba(16, 185, 129, 0.4);  
}

.brand-titles h1 {  
  font-size: 24px;  
  font-weight: 800;  
  background: linear-gradient(90deg, \#34d399, \#38bdf8);  
  \-webkit-background-clip: text;  
  \-webkit-text-fill-color: transparent;  
}

.brand-titles p {  
  font-size: 13px;  
  color: var(--text-muted);  
}

.header-badges {  
  display: flex;  
  gap: 10px;  
}

.badge {  
  display: inline-flex;  
  align-items: center;  
  padding: 4px 10px;  
  border-radius: 6px;  
  font-size: 12px;  
  font-weight: 600;  
  text-transform: uppercase;  
  letter-spacing: 0.5px;  
}

.badge-emerald { background: rgba(16, 185, 129, 0.15); color: \#34d399; border: 1px solid rgba(16, 185, 129, 0.3); }  
.badge-cyan { background: rgba(6, 182, 212, 0.15); color: \#38bdf8; border: 1px solid rgba(6, 182, 212, 0.3); }  
.badge-purple { background: rgba(139, 92, 246, 0.15); color: \#c084fc; border: 1px solid rgba(139, 92, 246, 0.3); }  
.badge-amber { background: rgba(245, 158, 11, 0.15); color: \#fbbf24; border: 1px solid rgba(245, 158, 11, 0.3); }  
.badge-red { background: rgba(239, 68, 68, 0.15); color: \#f87171; border: 1px solid rgba(239, 68, 68, 0.3); }

/\* Grid Layout \*/  
.studio-grid {  
  display: grid;  
  grid-template-columns: 500px 1fr;  
  gap: 24px;  
}

@media (max-width: 1140px) {  
  .studio-grid {  
    grid-template-columns: 1fr;  
  }  
}

.panel-card {  
  background-color: var(--bg-card);  
  border: 1px solid var(--border-color);  
  border-radius: 12px;  
  padding: 20px;  
  margin-bottom: 20px;  
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4);  
}

.panel-title {  
  font-size: 15px;  
  font-weight: 700;  
  color: \#e2e8f0;  
  margin-bottom: 16px;  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);  
  padding-bottom: 10px;  
}

/\* Forms \*/  
.form-group {  
  margin-bottom: 14px;  
}

.form-group label {  
  display: block;  
  font-size: 12px;  
  font-weight: 600;  
  color: var(--text-muted);  
  margin-bottom: 5px;  
}

.form-row {  
  display: grid;  
  grid-template-columns: 1fr 1fr;  
  gap: 12px;  
}

.form-row-3 {  
  display: grid;  
  grid-template-columns: 1fr 1fr 1fr;  
  gap: 10px;  
}

input, select {  
  width: 100%;  
  padding: 8px 12px;  
  background-color: \#070c1a;  
  border: 1px solid var(--border-color);  
  border-radius: 6px;  
  color: var(--text-main);  
  font-size: 13px;  
  font-family: inherit;  
  outline: none;  
  transition: border-color 0.2s;  
}

input:focus, select:focus {  
  border-color: var(--accent-emerald);  
}

.checkbox-label {  
  display: flex;  
  align-items: center;  
  gap: 8px;  
  font-size: 13px;  
  cursor: pointer;  
  user-select: none;  
  color: \#cbd5e1;  
}

.checkbox-label input\[type="checkbox"\] {  
  width: 16px;  
  height: 16px;  
  accent-color: var(--accent-emerald);  
}

.flags-grid {  
  display: grid;  
  grid-template-columns: repeat(2, 1fr);  
  gap: 10px;  
  background: \#080e1d;  
  padding: 12px;  
  border-radius: 8px;  
  border: 1px solid var(--border-color);  
  margin-bottom: 14px;  
}

.btn {  
  background: linear-gradient(135deg, \#059669, \#047857);  
  color: \#fff;  
  border: none;  
  padding: 8px 16px;  
  border-radius: 6px;  
  font-weight: 600;  
  font-size: 13px;  
  cursor: pointer;  
  display: inline-flex;  
  align-items: center;  
  gap: 6px;  
  transition: opacity 0.2s;  
}

.btn:hover { opacity: 0.9; }

.btn-secondary {  
  background: \#1e293b;  
  border: 1px solid var(--border-color);  
  color: \#cbd5e1;  
}

.btn-secondary:hover { background: \#334155; }

/\* Metrics Boxes \*/  
.metrics-row {  
  display: grid;  
  grid-template-columns: repeat(4, 1fr);  
  gap: 12px;  
  margin-bottom: 16px;  
}

.metric-box {  
  background: \#070c1a;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 12px;  
  text-align: center;  
}

.metric-val {  
  font-size: 19px;  
  font-weight: 800;  
  font-family: var(--font-mono);  
  color: \#34d399;  
}

.metric-val.alert-red { color: \#f87171; }  
.metric-val.alert-amber { color: \#fbbf24; }

.metric-lbl {  
  font-size: 11px;  
  color: var(--text-muted);  
  text-transform: uppercase;  
  margin-top: 4px;  
  letter-spacing: 0.5px;  
}

/\* SecTAG Byte Wireframe Dissection \*/  
.sectag-breakdown {  
  display: flex;  
  flex-wrap: wrap;  
  gap: 4px;  
  margin-bottom: 16px;  
  background: \#080e1d;  
  padding: 12px;  
  border-radius: 8px;  
  border: 1px solid var(--border-color);  
}

.byte-block {  
  padding: 6px 10px;  
  border-radius: 4px;  
  text-align: center;  
  font-family: var(--font-mono);  
}

.byte-block .byte-hex {  
  font-size: 13px;  
  font-weight: 800;  
}

.byte-block .byte-lbl {  
  font-size: 10px;  
  text-transform: uppercase;  
  color: rgba(255, 255, 255, 0.7);  
}

.bb-ethertype { background: rgba(59, 130, 246, 0.3); border: 1px solid \#3b82f6; color: \#93c5fd; }  
.bb-tci { background: rgba(16, 185, 129, 0.3); border: 1px solid \#10b981; color: \#6ee7b7; }  
.bb-sl { background: rgba(139, 92, 246, 0.3); border: 1px solid \#8b5cf6; color: \#c084fc; }  
.bb-pn { background: rgba(245, 158, 11, 0.3); border: 1px solid \#f59e0b; color: \#fde68a; }  
.bb-sci { background: rgba(6, 182, 212, 0.3); border: 1px solid \#06b6d4; color: \#67e8f9; }

/\* Audit Score Card \*/  
.audit-header-card {  
  display: flex;  
  align-items: center;  
  gap: 16px;  
  background: linear-gradient(135deg, \#09172e, \#0b2244);  
  border: 1px solid \#1e3a6a;  
  border-radius: 10px;  
  padding: 16px;  
  margin-bottom: 16px;  
}

.grade-badge-circle {  
  width: 58px;  
  height: 58px;  
  border-radius: 50%;  
  display: flex;  
  align-items: center;  
  justify-content: center;  
  font-size: 26px;  
  font-weight: 900;  
  font-family: var(--font-mono);  
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);  
  flex-shrink: 0;  
}

.grade-aplus { background: linear-gradient(135deg, \#10b981, \#059669); color: \#fff; }  
.grade-a { background: linear-gradient(135deg, \#34d399, \#10b981); color: \#000; }  
.grade-b { background: linear-gradient(135deg, \#38bdf8, \#0284c7); color: \#000; }  
.grade-c { background: linear-gradient(135deg, \#fbbf24, \#d97706); color: \#000; }  
.grade-d { background: linear-gradient(135deg, \#f97316, \#ea580c); color: \#fff; }  
.grade-f { background: linear-gradient(135deg, \#ef4444, \#b91c1c); color: \#fff; }

.audit-meta-area h3 {  
  font-size: 17px;  
  font-weight: 700;  
}

.audit-meta-area p {  
  font-size: 13px;  
  color: var(--text-muted);  
  margin-top: 2px;  
}

/\* Findings List \*/  
.findings-container {  
  display: flex;  
  flex-direction: column;  
  gap: 10px;  
}

.finding-item {  
  background: \#080e1d;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 12px 14px;  
}

.finding-top {  
  display: flex;  
  justify-content: space-between;  
  align-items: center;  
  margin-bottom: 6px;  
}

.finding-name {  
  font-size: 13px;  
  font-weight: 700;  
  color: \#e2e8f0;  
}

.finding-pts {  
  font-size: 12px;  
  font-family: var(--font-mono);  
  font-weight: 700;  
  padding: 2px 8px;  
  border-radius: 4px;  
}

.pts-pass { background: rgba(16, 185, 129, 0.15); color: \#34d399; }  
.pts-warn { background: rgba(245, 158, 11, 0.15); color: \#fbbf24; }  
.pts-crit { background: rgba(239, 68, 68, 0.15); color: \#f87171; }

.finding-desc {  
  font-size: 12px;  
  color: var(--text-muted);  
  line-height: 1.4;  
}

.finding-remed {  
  margin-top: 6px;  
  padding: 6px 10px;  
  background: rgba(245, 158, 11, 0.1);  
  border-left: 3px solid var(--accent-amber);  
  font-size: 12px;  
  color: \#fde68a;  
  border-radius: 0 4px 4px 0;  
}

/\* Code & Output Blocks \*/  
pre.code-block {  
  background-color: \#050811;  
  border: 1px solid var(--border-color);  
  border-radius: 8px;  
  padding: 14px;  
  font-family: var(--font-mono);  
  font-size: 12px;  
  color: \#cbd5e1;  
  overflow-x: auto;  
  line-height: 1.5;  
  white-space: pre;  
}

/\* Tabs \*/  
.tabs-nav {  
  display: flex;  
  gap: 6px;  
  margin-bottom: 12px;  
  border-bottom: 1px solid var(--border-color);  
  padding-bottom: 8px;  
}

.tab-btn {  
  background: transparent;  
  border: none;  
  color: var(--text-muted);  
  font-size: 13px;  
  font-weight: 600;  
  padding: 6px 12px;  
  border-radius: 6px;  
  cursor: pointer;  
  transition: all 0.2s;  
}

.tab-btn:hover {  
  color: var(--text-main);  
  background: rgba(255, 255, 255, 0.05);  
}

.tab-btn.active {  
  color: \#34d399;  
  background: rgba(16, 185, 129, 0.12);  
  border: 1px solid rgba(16, 185, 129, 0.3);  
}

.footer-bar {  
  text-align: center;  
  font-size: 12px;  
  color: var(--text-muted);  
  margin-top: 30px;  
  padding-top: 20px;  
  border-top: 1px solid var(--border-color);  
}  
