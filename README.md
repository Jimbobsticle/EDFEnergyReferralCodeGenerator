# EDFEnergyReferralCodeGenerator
# EDF Energy Referral Code Generator

<style>
  :root {
    --edf-orange: #FF6600;
    --edf-orange-light: #FF8533;
    --edf-orange-pale: #FFF0E6;
    --edf-dark: #1A1A2E;
    --edf-blue: #003366;
    --edf-blue-mid: #005299;
    --edf-blue-light: #E6EEF7;
    --edf-green: #00A651;
    --edf-green-pale: #E6F7EE;
    --edf-red: #c0392b;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  .app { max-width: 680px; margin: 0 auto; padding: 0 0 2rem; font-family: var(--font-sans); }
  .header {
    background: var(--edf-blue); color: white;
    padding: 1.25rem 1.5rem; border-radius: var(--border-radius-lg);
    margin-bottom: 1.5rem; display: flex; align-items: center; justify-content: space-between;
  }
  .header-left { display: flex; align-items: center; gap: 12px; }
  .edf-logo { background: var(--edf-orange); color: white; font-weight: 500; font-size: 14px; padding: 6px 10px; border-radius: 6px; letter-spacing: 0.5px; }
  .h-title { font-size: 17px; font-weight: 500; color: white; }
  .h-sub { font-size: 12px; color: rgba(255,255,255,0.65); margin-top: 2px; }
  .admin-btn { background: rgba(255,255,255,0.12); border: 0.5px solid rgba(255,255,255,0.3); color: white; padding: 6px 12px; border-radius: var(--border-radius-md); cursor: pointer; font-size: 13px; display: flex; align-items: center; gap: 6px; }
  .admin-btn:hover { background: rgba(255,255,255,0.2); }
  .stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-bottom: 1.5rem; }
  .stat-card { background: var(--color-background-secondary); border-radius: var(--border-radius-md); padding: 14px 16px; text-align: center; }
  .stat-num { font-size: 28px; font-weight: 500; color: var(--edf-orange); }
  .stat-label { font-size: 12px; color: var(--color-text-secondary); margin-top: 3px; }
  .card { background: var(--color-background-primary); border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-lg); padding: 1.25rem; margin-bottom: 1rem; }
  .card-title { font-size: 14px; font-weight: 500; color: var(--color-text-primary); margin-bottom: 1rem; display: flex; align-items: center; gap: 8px; }
  .card-title i { font-size: 18px; color: var(--edf-orange); }
  .input-row { display: flex; gap: 8px; }
  .input-row input { flex: 1; }
  input[type=text], input[type=password] { width: 100%; padding: 8px 12px; border: 0.5px solid var(--color-border-secondary); border-radius: var(--border-radius-md); font-size: 14px; background: var(--color-background-primary); color: var(--color-text-primary); }
  input:focus { outline: none; border-color: var(--edf-orange); box-shadow: 0 0 0 2px rgba(255,102,0,0.15); }
  .btn { padding: 8px 16px; border-radius: var(--border-radius-md); border: none; cursor: pointer; font-size: 14px; font-weight: 500; display: inline-flex; align-items: center; gap: 6px; transition: opacity 0.15s, transform 0.1s; }
  .btn:hover { opacity: 0.88; }
  .btn:active { transform: scale(0.98); }
  .btn-primary { background: var(--edf-orange); color: white; }
  .btn-blue { background: var(--edf-blue); color: white; }
  .btn-success { background: var(--edf-green); color: white; }
  .btn-ghost { background: var(--color-background-secondary); color: var(--color-text-primary); border: 0.5px solid var(--color-border-secondary); }
  .btn-danger { background: var(--edf-red); color: white; }
  .btn-warn { background: #e67e22; color: white; }
  .btn-sm { padding: 5px 10px; font-size: 12px; }
  .spin-section { text-align: center; padding: 0.5rem 0; }
  .spin-result { background: var(--edf-blue-light); border: 1px solid rgba(0,82,153,0.2); border-radius: var(--border-radius-lg); padding: 1.25rem; margin-top: 1rem; display: none; }
  .spin-result.show { display: block; }
  .spin-url { font-size: 13px; color: var(--edf-blue-mid); word-break: break-all; margin: 8px 0; font-family: var(--font-mono); }
  .spin-actions { display: flex; gap: 8px; justify-content: center; margin-top: 10px; flex-wrap: wrap; }
  .list-item { display: flex; align-items: center; gap: 10px; padding: 10px 12px; border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-md); margin-bottom: 6px; font-size: 13px; }
  .list-item-url { flex: 1; color: var(--edf-blue-mid); overflow: hidden; text-overflow: ellipsis; white-space: nowrap; font-family: var(--font-mono); font-size: 12px; }
  .list-item-meta { font-size: 11px; color: var(--color-text-secondary); white-space: nowrap; }
  .expiry-badge { font-size: 10px; padding: 1px 7px; border-radius: 10px; white-space: nowrap; }
  .exp-ok { background: var(--edf-green-pale); color: var(--edf-green); }
  .exp-soon { background: #FFF3E0; color: #e67e22; }
  .exp-old { background: #FEECEB; color: var(--edf-red); }
  .flagged-badge { font-size: 10px; padding: 1px 7px; border-radius: 10px; background: #FEECEB; color: var(--edf-red); }
  .toast { position: fixed; top: 12px; right: 12px; background: var(--edf-dark); color: white; padding: 10px 16px; border-radius: var(--border-radius-md); font-size: 13px; z-index: 99; opacity: 0; pointer-events: none; display: flex; align-items: center; gap: 8px; transition: opacity 0.25s; }
  .toast.show { opacity: 1; pointer-events: auto; }
  .overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.45); z-index: 50; display: none; align-items: center; justify-content: center; }
  .overlay.show { display: flex; }
  .modal { background: var(--color-background-primary); border-radius: var(--border-radius-lg); padding: 1.5rem; width: 90%; max-width: 500px; border: 0.5px solid var(--color-border-secondary); max-height: 88vh; overflow-y: auto; }
  .modal h2 { font-size: 16px; font-weight: 500; margin-bottom: 1rem; display: flex; align-items: center; gap: 8px; }
  .modal h2 i { color: var(--edf-orange); }
  .modal-footer { display: flex; gap: 8px; justify-content: flex-end; margin-top: 1rem; }
  .empty-state { text-align: center; padding: 2rem 1rem; color: var(--color-text-secondary); font-size: 14px; }
  .empty-state i { font-size: 40px; color: var(--color-border-tertiary); display: block; margin-bottom: 8px; }
  .admin-stat-row { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; margin-bottom: 1rem; }
  .admin-stat { background: var(--edf-blue-light); border-radius: var(--border-radius-md); padding: 10px; text-align: center; }
  .admin-stat .n { font-size: 22px; font-weight: 500; color: var(--edf-blue); }
  .admin-stat .l { font-size: 11px; color: var(--edf-blue-mid); margin-top: 2px; }
  .rate-bar { height: 3px; border-radius: 2px; background: var(--edf-orange); margin-top: 4px; transition: width 0.3s; }

  .slot-machine { display: flex; justify-content: center; margin: 12px 0 4px; height: 36px; overflow: hidden; position: relative; }
  .slot-reel { display: flex; flex-direction: column; align-items: center; }
  .slot-item { font-size: 13px; font-family: var(--font-mono); color: var(--edf-blue-mid); height: 36px; display: flex; align-items: center; padding: 0 8px; white-space: nowrap; overflow: hidden; max-width: 420px; text-overflow: ellipsis; }
  @keyframes reelSpin {
    0%   { transform: translateY(0); }
    100% { transform: translateY(-720px); }
  }
  @keyframes reelSettle {
    0%   { transform: translateY(-720px); }
    60%  { transform: translateY(6px); }
    80%  { transform: translateY(-3px); }
    100% { transform: translateY(0); }
  }
  .spinning .slot-reel { animation: reelSpin 0.6s linear; }
  .settling .slot-reel { animation: reelSettle 0.4s ease-out forwards; }

  .report-form textarea { width: 100%; padding: 8px 12px; border: 0.5px solid var(--color-border-secondary); border-radius: var(--border-radius-md); font-size: 13px; background: var(--color-background-primary); color: var(--color-text-primary); resize: vertical; min-height: 70px; font-family: var(--font-sans); }
  .info-note { font-size: 12px; color: var(--color-text-secondary); background: var(--color-background-secondary); border-radius: var(--border-radius-md); padding: 8px 12px; margin-top: 8px; display: flex; gap: 6px; align-items: flex-start; }
  .adm-link-row { border: 0.5px solid var(--color-border-tertiary); border-radius: var(--border-radius-md); padding: 10px 12px; margin-bottom: 6px; font-size: 13px; }
  .adm-link-url { font-family: var(--font-mono); font-size: 11px; color: var(--edf-blue-mid); word-break: break-all; margin-bottom: 4px; }
  .adm-link-meta { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
  .adm-flag { background: #FEECEB; color: var(--edf-red); font-size: 11px; padding: 1px 7px; border-radius: 10px; }
  .adm-flag-reason { font-size: 11px; color: var(--edf-red); font-style: italic; margin-top: 3px; }
  .adm-actions { display: flex; gap: 6px; margin-top: 6px; flex-wrap: wrap; }
  .tab-row { display: flex; gap: 6px; margin-bottom: 1rem; }
  .tab { padding: 6px 14px; border-radius: var(--border-radius-md); font-size: 13px; cursor: pointer; border: 0.5px solid var(--color-border-secondary); background: var(--color-background-secondary); color: var(--color-text-secondary); }
  .tab.active { background: var(--edf-blue); color: white; border-color: var(--edf-blue); }
</style>

<div class="app">
  <h2 class="sr-only" style="position:absolute;width:1px;height:1px;overflow:hidden;clip:rect(0,0,0,0);">EDF Energy referral code generator — submit, discover and share referral links</h2>

  <div class="header">
    <div class="header-left">
      <div class="edf-logo">EDF</div>
      <div>
        <div class="h-title">Referral Code Generator</div>
        <div class="h-sub">Share &amp; discover EDF energy referral links</div>
      </div>
    </div>
    <button class="admin-btn" onclick="openAdminLogin()"><i class="ti ti-lock" aria-hidden="true"></i> Admin</button>
  </div>

  <div class="stats-grid">
    <div class="stat-card"><div class="stat-num" id="stat-submitted">0</div><div class="stat-label">Codes submitted</div></div>
    <div class="stat-card"><div class="stat-num" id="stat-generated">0</div><div class="stat-label">Times generated</div></div>
    <div class="stat-card"><div class="stat-num" id="stat-clicked">0</div><div class="stat-label">Links clicked</div></div>
  </div>

  <div class="card">
    <div class="card-title"><i class="ti ti-link" aria-hidden="true"></i> Submit your referral link</div>
    <div class="input-row">
      <input type="text" id="submitUrl" placeholder="https://refer.edfenergy.com/...">
      <button class="btn btn-primary" onclick="submitLink()"><i class="ti ti-plus" aria-hidden="true"></i> Submit</button>
    </div>
    <input type="text" id="submitNick" placeholder="Your nickname (optional)" style="margin-top:8px;">
    <div id="submitMsg" style="margin-top:8px;font-size:13px;display:none;"></div>
    <div id="rateLimitBar" style="display:none;margin-top:10px;">
      <div style="display:flex;justify-content:space-between;font-size:11px;color:var(--color-text-secondary);"><span>Daily submission limit</span><span id="rateLimitText"></span></div>
      <div class="rate-bar" id="rateBarFill" style="width:0%;"></div>
    </div>
    <div class="info-note"><i class="ti ti-info-circle" style="font-size:14px;flex-shrink:0;margin-top:1px;" aria-hidden="true"></i> Only official edfenergy.com referral links are accepted. Links expire after 12 months and are automatically removed.</div>
  </div>

  <div class="card">
    <div class="card-title"><i class="ti ti-refresh" aria-hidden="true"></i> Get a random referral link</div>
    <div class="spin-section">
      <div class="slot-machine" id="slotMachine" aria-live="polite">
        <div class="slot-reel" id="slotReel"><div class="slot-item" id="slotDisplay" style="color:var(--color-text-secondary);">Press generate to spin a link</div></div>
      </div>
      <button class="btn btn-blue" onclick="spinLink()" id="spinBtn" style="padding:10px 28px;font-size:15px;margin-top:8px;">
        <i class="ti ti-player-play" aria-hidden="true"></i> Generate random link
      </button>
    </div>
    <div class="spin-result" id="spinResult">
      <div style="font-size:12px;color:var(--color-text-secondary);">Selected link</div>
      <div class="spin-url" id="spinUrl"></div>
      <div style="font-size:12px;color:var(--color-text-secondary);margin-top:2px;" id="spinNick"></div>
      <div class="spin-actions">
        <button class="btn btn-success btn-sm" onclick="copySpun()"><i class="ti ti-copy" aria-hidden="true"></i> Copy link</button>
        <button class="btn btn-primary btn-sm" onclick="openSpun()"><i class="ti ti-external-link" aria-hidden="true"></i> Open link</button>
        <button class="btn btn-ghost btn-sm" onclick="shareSpun()"><i class="ti ti-share" aria-hidden="true"></i> Share</button>
        <button class="btn btn-warn btn-sm" onclick="promptReport()"><i class="ti ti-flag" aria-hidden="true"></i> Report</button>
      </div>
    </div>
  </div>

  <div class="card">
    <div class="card-title"><i class="ti ti-list" aria-hidden="true"></i> Submitted links <span style="font-size:12px;font-weight:400;color:var(--color-text-secondary);margin-left:4px;">— use your PIN to remove your own</span></div>
    <div id="linkList"></div>
  </div>
</div>

<div class="overlay" id="removeOverlay">
  <div class="modal">
    <h2><i class="ti ti-trash" aria-hidden="true"></i> Remove your link</h2>
    <p style="font-size:14px;color:var(--color-text-secondary);margin-bottom:1rem;">Enter the 4-digit PIN you received when submitting to confirm removal.</p>
    <input type="password" id="removePinInput" placeholder="Your 4-digit PIN">
    <div id="removePinMsg" style="margin-top:6px;font-size:13px;display:none;"></div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="closeModal('removeOverlay')">Cancel</button>
      <button class="btn btn-danger" onclick="confirmRemove()"><i class="ti ti-trash" aria-hidden="true"></i> Remove</button>
    </div>
  </div>
</div>

<div class="overlay" id="reportOverlay">
  <div class="modal">
    <h2><i class="ti ti-flag" aria-hidden="true"></i> Report this link</h2>
    <p style="font-size:14px;color:var(--color-text-secondary);margin-bottom:1rem;">Please tell us why you're reporting this link. An admin will review it.</p>
    <div class="report-form">
      <textarea id="reportReason" placeholder="Describe the issue (e.g. broken link, spam, not an EDF link)..."></textarea>
    </div>
    <div id="reportMsg" style="margin-top:6px;font-size:13px;display:none;"></div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="closeModal('reportOverlay')">Cancel</button>
      <button class="btn btn-warn" onclick="submitReport()"><i class="ti ti-flag" aria-hidden="true"></i> Submit report</button>
    </div>
  </div>
</div>

<div class="overlay" id="adminLoginOverlay">
  <div class="modal">
    <h2><i class="ti ti-lock" aria-hidden="true"></i> Admin access</h2>
    <p style="font-size:14px;color:var(--color-text-secondary);margin-bottom:1rem;">Enter the admin password to manage submitted links.</p>
    <input type="password" id="adminPassInput" placeholder="Admin password">
    <div id="adminLoginMsg" style="margin-top:6px;font-size:13px;display:none;"></div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="closeModal('adminLoginOverlay')">Cancel</button>
      <button class="btn btn-primary" onclick="checkAdminPass()"><i class="ti ti-arrow-right" aria-hidden="true"></i> Enter</button>
    </div>
  </div>
</div>

<div class="overlay" id="adminPanelOverlay">
  <div class="modal" style="max-width:580px;">
    <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:1rem;">
      <h2 style="margin:0;"><i class="ti ti-shield-check" aria-hidden="true"></i> Admin panel</h2>
      <button class="btn btn-ghost btn-sm" onclick="closeModal('adminPanelOverlay')"><i class="ti ti-x" aria-hidden="true"></i></button>
    </div>
    <div class="admin-stat-row">
      <div class="admin-stat"><div class="n" id="adm-submitted">0</div><div class="l">Active links</div></div>
      <div class="admin-stat"><div class="n" id="adm-generated">0</div><div class="l">Generated</div></div>
      <div class="admin-stat"><div class="n" id="adm-clicked">0</div><div class="l">Clicked</div></div>
    </div>
    <div class="tab-row">
      <div class="tab active" id="tabAll" onclick="adminTab('all')">All links</div>
      <div class="tab" id="tabFlagged" onclick="adminTab('flagged')">Flagged <span id="flagCount"></span></div>
      <div class="tab" id="tabExpired" onclick="adminTab('expired')">Expired</div>
    </div>
    <div id="adminLinkList"></div>
    <div style="margin-top:1rem;padding-top:1rem;border-top:0.5px solid var(--color-border-tertiary);">
      <p style="font-size:11px;color:var(--color-text-secondary);">Change admin password via the <code>ADMIN_PASS</code> constant in the source.</p>
    </div>
  </div>
</div>

<div class="toast" id="toast"><i class="ti ti-check" aria-hidden="true" id="toastIcon"></i> <span id="toastMsg"></span></div>

<script>
const ADMIN_PASS = "5jkhfHH2-edBm!@fdj4";
const MAX_DAILY_SUBMITS = 3;
const EXPIRE_MS = 365 * 24 * 60 * 60 * 1000;
const EDF_PATTERN = /^https?:\/\/([\w-]+\.)*edfenergy\.com(\/|$)/i;

let links = [];
let globalStats = { submitted: 0, generated: 0, clicked: 0 };
let pendingRemoveIndex = -1;
let pendingReportIndex = -1;
let currentSpunUrl = "";
let adminTabMode = "all";

function getTodayKey() { return "edf_rl_" + new Date().toISOString().slice(0, 10); }

function getRateCount() {
  try { return parseInt(localStorage.getItem(getTodayKey()) || "0"); } catch(e) { return 0; }
}
function incRateCount() {
  try { localStorage.setItem(getTodayKey(), (getRateCount() + 1).toString()); } catch(e) {}
}

function isExpired(link) { return Date.now() - link.submitted > EXPIRE_MS; }
function expiresInDays(link) { return Math.ceil((link.submitted + EXPIRE_MS - Date.now()) / 86400000); }

async function loadData() {
  try { const r = await window.storage.get("edf_links_v2", true); if (r) links = JSON.parse(r.value); } catch(e) { links = []; }
  try { const s = await window.storage.get("edf_stats_v2", true); if (s) globalStats = JSON.parse(s.value); } catch(e) {}
  purgeExpired();
  renderStats(); renderList(); updateRateBar();
}

async function saveLinks() { try { await window.storage.set("edf_links_v2", JSON.stringify(links), true); } catch(e) {} }
async function saveStats() { try { await window.storage.set("edf_stats_v2", JSON.stringify(globalStats), true); } catch(e) {} }

function purgeExpired() {
  const before = links.length;
  links = links.filter(l => !isExpired(l));
  if (links.length !== before) saveLinks();
}

function renderStats() {
  const active = links.filter(l => !isExpired(l));
  document.getElementById("stat-submitted").textContent = active.length;
  document.getElementById("stat-generated").textContent = globalStats.generated;
  document.getElementById("stat-clicked").textContent = globalStats.clicked;
}

function expiryBadge(link) {
  const d = expiresInDays(link);
  if (d <= 30) return `<span class="expiry-badge exp-soon">Expires in ${d}d</span>`;
  return `<span class="expiry-badge exp-ok">Expires in ${d}d</span>`;
}

function renderList() {
  const el = document.getElementById("linkList");
  const active = links.filter(l => !isExpired(l));
  if (!active.length) {
    el.innerHTML = '<div class="empty-state"><i class="ti ti-link-off" aria-hidden="true"></i>No links submitted yet. Be the first!</div>';
    return;
  }
  el.innerHTML = active.map((l, i) => {
    const realIdx = links.indexOf(l);
    return `<div class="list-item">
      <i class="ti ti-bolt" style="color:var(--edf-orange);font-size:16px;flex-shrink:0;" aria-hidden="true"></i>
      <div style="flex:1;min-width:0;">
        <div class="list-item-url">${l.url}</div>
        <div style="display:flex;gap:6px;align-items:center;margin-top:3px;flex-wrap:wrap;">
          ${l.nick ? `<span style="font-size:11px;color:var(--color-text-secondary);">${l.nick}</span>` : ""}
          ${expiryBadge(l)}
          ${l.flagged ? `<span class="flagged-badge"><i class="ti ti-flag" style="font-size:10px;" aria-hidden="true"></i> Reported</span>` : ""}
        </div>
      </div>
      <div class="list-item-meta" style="display:flex;flex-direction:column;align-items:flex-end;gap:4px;">
        <span style="font-size:11px;"><i class="ti ti-mouse" style="font-size:11px;" aria-hidden="true"></i> ${l.clicks||0}</span>
      </div>
      <button class="btn btn-ghost btn-sm" onclick="promptRemove(${realIdx})" title="Remove your link" aria-label="Remove link"><i class="ti ti-trash" style="color:var(--edf-red);" aria-hidden="true"></i></button>
    </div>`;
  }).join("");
}

function updateRateBar() {
  const count = getRateCount();
  const pct = Math.min(100, Math.round((count / MAX_DAILY_SUBMITS) * 100));
  document.getElementById("rateLimitBar").style.display = "block";
  document.getElementById("rateLimitText").textContent = `${count}/${MAX_DAILY_SUBMITS} today`;
  document.getElementById("rateBarFill").style.width = pct + "%";
  document.getElementById("rateBarFill").style.background = count >= MAX_DAILY_SUBMITS ? "var(--edf-red)" : "var(--edf-orange)";
}

function submitLink() {
  const url = document.getElementById("submitUrl").value.trim();
  const nick = document.getElementById("submitNick").value.trim();
  const msg = document.getElementById("submitMsg");
  msg.style.display = "none";
  if (!url) { showMsg(msg, "Please enter a URL.", "var(--edf-red)"); return; }
  if (!EDF_PATTERN.test(url)) { showMsg(msg, "Only official edfenergy.com referral links are accepted. Example: https://refer.edfenergy.com/...", "var(--edf-red)"); return; }
  if (getRateCount() >= MAX_DAILY_SUBMITS) { showMsg(msg, `Daily limit reached (${MAX_DAILY_SUBMITS} submissions per day). Please try again tomorrow.`, "var(--edf-red)"); return; }
  if (links.find(l => l.url.toLowerCase() === url.toLowerCase())) { showMsg(msg, "This URL has already been submitted.", "var(--edf-red)"); return; }
  const pin = Math.floor(1000 + Math.random() * 9000).toString();
  links.push({ url, nick: nick||"", pin, clicks: 0, generated: 0, submitted: Date.now(), flagged: false, flagReason: "" });
  globalStats.submitted = links.length;
  incRateCount();
  saveLinks(); saveStats(); renderStats(); renderList(); updateRateBar();
  document.getElementById("submitUrl").value = "";
  document.getElementById("submitNick").value = "";
  showMsg(msg, `Submitted! Your removal PIN is: ${pin} — write this down, it cannot be recovered.`, "var(--edf-green)");
  showToast("Link submitted successfully", "ti-check");
}

function showMsg(el, text, color) { el.textContent = text; el.style.color = color; el.style.display = "block"; }

function spinLink() {
  const active = links.filter(l => !isExpired(l) && !l.flagged);
  if (!active.length) { showToast("No links available", "ti-alert-circle"); return; }

  const btn = document.getElementById("spinBtn");
  btn.disabled = true;
  const sm = document.getElementById("slotMachine");
  const reel = document.getElementById("slotReel");
  const chosen = active[Math.floor(Math.random() * active.length)];

  const dummy = Array.from({length: 20}, (_, i) => active[i % active.length].url);
  reel.innerHTML = dummy.map(u => `<div class="slot-item">${u}</div>`).join("") +
    `<div class="slot-item" style="color:var(--edf-blue);font-weight:500;">${chosen.url}</div>`;

  sm.classList.remove("settling");
  sm.classList.add("spinning");
  setTimeout(() => {
    sm.classList.remove("spinning");
    sm.classList.add("settling");
    setTimeout(() => {
      reel.innerHTML = `<div class="slot-item" style="color:var(--edf-blue);font-weight:500;">${chosen.url}</div>`;
      sm.classList.remove("settling");
      btn.disabled = false;
    }, 450);
  }, 620);

  const realIdx = links.indexOf(chosen);
  links[realIdx].generated = (links[realIdx].generated||0) + 1;
  globalStats.generated++;
  saveLinks(); saveStats(); renderStats();
  currentSpunUrl = chosen.url;

  const res = document.getElementById("spinResult");
  document.getElementById("spinUrl").textContent = chosen.url;
  document.getElementById("spinNick").textContent = chosen.nick ? `Submitted by: ${chosen.nick}` : "";
  res.classList.add("show");
}

function copySpun() {
  if (!currentSpunUrl) return;
  navigator.clipboard.writeText(currentSpunUrl).then(() => showToast("Link copied to clipboard!", "ti-copy"));
  recordClick();
}

function openSpun() {
  if (!currentSpunUrl) return;
  recordClick();
  window.open(currentSpunUrl, "_blank");
}

function shareSpun() {
  if (!currentSpunUrl) return;
  if (navigator.share) {
    navigator.share({ title: "EDF Energy referral link", text: "Use this EDF referral link to save on your energy bill:", url: currentSpunUrl })
      .then(() => { recordClick(); showToast("Shared!", "ti-share"); })
      .catch(() => {});
  } else {
    navigator.clipboard.writeText(currentSpunUrl).then(() => showToast("Link copied — paste to share!", "ti-copy"));
    recordClick();
  }
}

function recordClick() {
  const idx = links.findIndex(l => l.url === currentSpunUrl);
  if (idx >= 0) { links[idx].clicks = (links[idx].clicks||0) + 1; globalStats.clicked++; saveLinks(); saveStats(); renderStats(); }
}

function promptRemove(i) {
  pendingRemoveIndex = i;
  document.getElementById("removePinInput").value = "";
  const m = document.getElementById("removePinMsg"); m.style.display = "none";
  document.getElementById("removeOverlay").classList.add("show");
}

function confirmRemove() {
  const pin = document.getElementById("removePinInput").value.trim();
  const msg = document.getElementById("removePinMsg");
  if (!pin) { showMsg(msg, "Please enter your PIN.", "var(--edf-red)"); return; }
  if (pendingRemoveIndex < 0 || pendingRemoveIndex >= links.length) return;
  if (links[pendingRemoveIndex].pin !== pin) { showMsg(msg, "Incorrect PIN. Only the original submitter can remove this link.", "var(--edf-red)"); return; }
  links.splice(pendingRemoveIndex, 1);
  saveLinks(); renderList(); renderStats();
  closeModal("removeOverlay");
  showToast("Link removed", "ti-check");
}

function promptReport() {
  if (!currentSpunUrl) return;
  pendingReportIndex = links.findIndex(l => l.url === currentSpunUrl);
  document.getElementById("reportReason").value = "";
  const m = document.getElementById("reportMsg"); m.style.display = "none";
  document.getElementById("reportOverlay").classList.add("show");
}

function submitReport() {
  const reason = document.getElementById("reportReason").value.trim();
  const msg = document.getElementById("reportMsg");
  if (!reason) { showMsg(msg, "Please describe the issue.", "var(--edf-red)"); return; }
  if (pendingReportIndex >= 0 && pendingReportIndex < links.length) {
    links[pendingReportIndex].flagged = true;
    links[pendingReportIndex].flagReason = reason;
    saveLinks(); renderList();
  }
  closeModal("reportOverlay");
  showToast("Report submitted — an admin will review it", "ti-flag");
}

function openAdminLogin() {
  document.getElementById("adminPassInput").value = "";
  const m = document.getElementById("adminLoginMsg"); m.style.display = "none";
  document.getElementById("adminLoginOverlay").classList.add("show");
}

function checkAdminPass() {
  if (document.getElementById("adminPassInput").value === ADMIN_PASS) {
    closeModal("adminLoginOverlay"); openAdminPanel();
  } else {
    showMsg(document.getElementById("adminLoginMsg"), "Incorrect password.", "var(--edf-red)");
  }
}

function adminTab(mode) {
  adminTabMode = mode;
  ["all","flagged","expired"].forEach(t => {
    document.getElementById("tab" + t.charAt(0).toUpperCase() + t.slice(1)).classList.toggle("active", t === mode);
  });
  renderAdminList();
}

function openAdminPanel() {
  document.getElementById("adm-submitted").textContent = links.filter(l => !isExpired(l)).length;
  document.getElementById("adm-generated").textContent = globalStats.generated;
  document.getElementById("adm-clicked").textContent = globalStats.clicked;
  const flagged = links.filter(l => l.flagged).length;
  document.getElementById("flagCount").textContent = flagged ? `(${flagged})` : "";
  adminTabMode = "all";
  document.getElementById("tabAll").classList.add("active");
  document.getElementById("tabFlagged").classList.remove("active");
  document.getElementById("tabExpired").classList.remove("active");
  renderAdminList();
  document.getElementById("adminPanelOverlay").classList.add("show");
}

function renderAdminList() {
  const el = document.getElementById("adminLinkList");
  let subset = links;
  if (adminTabMode === "flagged") subset = links.filter(l => l.flagged);
  else if (adminTabMode === "expired") subset = links.filter(l => isExpired(l));
  if (!subset.length) { el.innerHTML = '<div class="empty-state" style="padding:1rem 0;"><i class="ti ti-link-off" aria-hidden="true"></i>Nothing here.</div>'; return; }
  el.innerHTML = subset.map(l => {
    const realIdx = links.indexOf(l);
    const d = expiresInDays(l);
    const expLabel = isExpired(l) ? `<span class="expiry-badge exp-old">Expired</span>` : (d <= 30 ? `<span class="expiry-badge exp-soon">${d}d left</span>` : `<span class="expiry-badge exp-ok">${d}d left</span>`);
    return `<div class="adm-link-row">
      <div class="adm-link-url">${l.url}</div>
      ${l.flagReason ? `<div class="adm-flag-reason"><i class="ti ti-flag" style="font-size:11px;" aria-hidden="true"></i> "${l.flagReason}"</div>` : ""}
      <div class="adm-link-meta">
        ${l.nick ? `<span style="font-size:11px;color:var(--color-text-secondary);">${l.nick}</span>` : ""}
        ${expLabel}
        ${l.flagged ? `<span class="adm-flag"><i class="ti ti-flag" style="font-size:10px;" aria-hidden="true"></i> Flagged</span>` : ""}
        <span style="font-size:11px;color:var(--color-text-secondary);"><i class="ti ti-refresh" style="font-size:11px;" aria-hidden="true"></i> ${l.generated||0} gen</span>
        <span style="font-size:11px;color:var(--color-text-secondary);"><i class="ti ti-mouse" style="font-size:11px;" aria-hidden="true"></i> ${l.clicks||0} clicks</span>
      </div>
      <div class="adm-actions">
        <button class="btn btn-danger btn-sm" onclick="adminRemove(${realIdx})"><i class="ti ti-trash" aria-hidden="true"></i> Remove</button>
        ${l.flagged ? `<button class="btn btn-success btn-sm" onclick="adminClearFlag(${realIdx})"><i class="ti ti-check" aria-hidden="true"></i> Clear flag</button>` : ""}
      </div>
    </div>`;
  }).join("");
}

function adminRemove(i) { links.splice(i, 1); saveLinks(); renderList(); renderStats(); openAdminPanel(); }
function adminClearFlag(i) { links[i].flagged = false; links[i].flagReason = ""; saveLinks(); renderList(); openAdminPanel(); }

function closeModal(id) { document.getElementById(id).classList.remove("show"); }

let toastTimer;
function showToast(msg, icon) {
  document.getElementById("toastMsg").textContent = msg;
  document.getElementById("toastIcon").className = `ti ti-${icon||"check"}`;
  const t = document.getElementById("toast");
  t.classList.add("show");
  clearTimeout(toastTimer);
  toastTimer = setTimeout(() => t.classList.remove("show"), 2800);
}

document.getElementById("adminPassInput").addEventListener("keydown", e => { if (e.key==="Enter") checkAdminPass(); });
document.getElementById("submitUrl").addEventListener("keydown", e => { if (e.key==="Enter") submitLink(); });
document.getElementById("removePinInput").addEventListener("keydown", e => { if (e.key==="Enter") confirmRemove(); });

loadData();
</script>
