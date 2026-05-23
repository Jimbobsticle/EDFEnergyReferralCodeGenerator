<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>r/EDFEnergy Referral Code Generator</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@latest/tabler-icons.min.css">

<!-- Firebase -->
<script type="module">
  import { initializeApp } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js";
  import {
    getFirestore, collection, doc, getDocs, addDoc, updateDoc, deleteDoc,
    query, orderBy, onSnapshot, increment, getDoc, setDoc
  } from "https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore.js";

  // ============================================================
  // PASTE YOUR FIREBASE CONFIG HERE (from Firebase console)
  // ============================================================
  const firebaseConfig = {
    apiKey:            "AIzaSyD-POLJzNpSIl4SnHiex0eLEFLksd_PnG4",
    authDomain:        "edf-referral-code-generator.firebaseapp.com",
    projectId:         "edf-referral-code-generator",
    storageBucket:     "edf-referral-code-generator.firebasestorage.app",
    messagingSenderId: "746012825348",
    appId:             "1:746012825348:web:8e73e49eda96099fbc2251"
  };
  // ============================================================

  const ADMIN_PASS       = "5jkhfHH2-edBm!@fdj4";
  const MAX_DAILY        = 3;
  const EXPIRE_MS        = 365 * 24 * 60 * 60 * 1000;
  const EDF_PATTERN      = /^https?:\/\/([\w-]+\.)*edfenergy\.com(\/|$)/i;

  const app = initializeApp(firebaseConfig);
  const db  = getFirestore(app);

  let links            = [];   // local cache
  let globalStats      = { generated: 0, clicked: 0 };
  let pendingRemoveId  = null;
  let pendingReportId  = null;
  let currentSpunUrl   = "";
  let currentSpunId    = null;
  let adminTabMode     = "all";

  // ── Rate limit (localStorage, per-browser) ──────────────────
  function getTodayKey() { return "edf_rl_" + new Date().toISOString().slice(0,10); }
  function getRateCount() { try { return parseInt(localStorage.getItem(getTodayKey())||"0"); } catch(e){ return 0; } }
  function incRateCount() { try { localStorage.setItem(getTodayKey(), (getRateCount()+1).toString()); } catch(e){} }

  // ── Helpers ──────────────────────────────────────────────────
  function isExpired(l)       { return Date.now() - l.submitted > EXPIRE_MS; }
  function expiresInDays(l)   { return Math.ceil((l.submitted + EXPIRE_MS - Date.now()) / 86400000); }

  function showMsg(el, text, color) { el.textContent=text; el.style.color=color; el.style.display="block"; }

  let toastTimer;
  function showToast(msg, icon) {
    document.getElementById("toastMsg").textContent = msg;
    document.getElementById("toastIcon").className  = `ti ti-${icon||"check"}`;
    const t = document.getElementById("toast");
    t.classList.add("show");
    clearTimeout(toastTimer);
    toastTimer = setTimeout(()=>t.classList.remove("show"), 2800);
  }

  function closeModal(id) { document.getElementById(id).classList.remove("show"); }

  // ── Stats doc ────────────────────────────────────────────────
  const statsRef = doc(db, "stats", "global");

  async function loadStats() {
    try {
      const snap = await getDoc(statsRef);
      if (snap.exists()) globalStats = snap.data();
      else await setDoc(statsRef, { generated:0, clicked:0 });
    } catch(e) { console.error("loadStats:", e); }
    renderStats();
  }

  async function bumpStat(field) {
    globalStats[field] = (globalStats[field]||0) + 1;
    renderStats();
    try { await updateDoc(statsRef, { [field]: increment(1) }); } catch(e){}
  }

  // ── Links collection ─────────────────────────────────────────
  function listenLinks() {
    const q = query(collection(db,"links"), orderBy("submitted","desc"));
    onSnapshot(q, snap => {
      links = snap.docs.map(d => ({ id:d.id, ...d.data() }));
      renderStats(); renderList();
    }, err => console.error("listenLinks:", err));
  }

  function renderStats() {
    const active = links.filter(l => !isExpired(l));
    document.getElementById("stat-submitted").textContent = active.length;
    document.getElementById("stat-generated").textContent = globalStats.generated||0;
    document.getElementById("stat-clicked").textContent   = globalStats.clicked||0;
  }

  function expiryBadge(l) {
    if (isExpired(l)) return `<span class="expiry-badge exp-old">Expired</span>`;
    const d = expiresInDays(l);
    return d<=30
      ? `<span class="expiry-badge exp-soon">Expires in ${d}d</span>`
      : `<span class="expiry-badge exp-ok">Expires in ${d}d</span>`;
  }

  function renderList() {
    const el = document.getElementById("linkList");
    const active = links.filter(l => !isExpired(l));
    if (!active.length) {
      el.innerHTML = '<div class="empty-state"><i class="ti ti-link-off"></i>No links submitted yet. Be the first!</div>';
      return;
    }
    el.innerHTML = active.map(l => `
      <div class="list-item">
        <i class="ti ti-bolt" style="color:var(--edf-orange);font-size:16px;flex-shrink:0;"></i>
        <div style="flex:1;min-width:0;">
          <div class="list-item-url">${l.url}</div>
          <div style="display:flex;gap:6px;align-items:center;margin-top:3px;flex-wrap:wrap;">
            ${l.nick ? `<span style="font-size:11px;color:var(--clr-text-muted);">${l.nick}</span>` : ""}
            ${expiryBadge(l)}
            ${l.flagged ? `<span class="flagged-badge"><i class="ti ti-flag" style="font-size:10px;"></i> Reported</span>` : ""}
          </div>
        </div>
        <span style="font-size:11px;color:var(--clr-text-muted);white-space:nowrap;">
          <i class="ti ti-mouse" style="font-size:11px;"></i> ${l.clicks||0}
        </span>
        <button class="btn btn-ghost btn-sm" onclick="window._promptRemove('${l.id}')" title="Remove your link">
          <i class="ti ti-trash" style="color:#e74c3c;"></i>
        </button>
      </div>`).join("");
  }

  // ── Submit ───────────────────────────────────────────────────
  async function submitLink() {
    const url  = document.getElementById("submitUrl").value.trim();
    const nick = document.getElementById("submitNick").value.trim();
    const msg  = document.getElementById("submitMsg");
    msg.style.display = "none";

    if (!url)                        { showMsg(msg,"Please enter a URL.","#e74c3c"); return; }
    if (!EDF_PATTERN.test(url))      { showMsg(msg,"Only official edfenergy.com referral links are accepted. Example: https://refer.edfenergy.com/...","#e74c3c"); return; }
    if (getRateCount()>=MAX_DAILY)   { showMsg(msg,`Daily limit reached (${MAX_DAILY} submissions per day). Please try again tomorrow.`,"#e74c3c"); return; }
    if (links.find(l=>l.url.toLowerCase()===url.toLowerCase())) { showMsg(msg,"This URL has already been submitted.","#e74c3c"); return; }

    const pin = Math.floor(1000+Math.random()*9000).toString();
    try {
      await addDoc(collection(db,"links"), {
        url, nick: nick||"", pin, clicks:0, generated:0,
        submitted: Date.now(), flagged:false, flagReason:""
      });
      incRateCount();
      document.getElementById("submitUrl").value  = "";
      document.getElementById("submitNick").value = "";
      showMsg(msg,`Submitted! Your removal PIN is: ${pin} — write this down, it cannot be recovered.`,"#00A651");
      showToast("Link submitted successfully","check");
      updateRateBar();
    } catch(e) { showMsg(msg,"Error saving link — please try again.","#e74c3c"); console.error(e); }
  }

  // ── Spin ─────────────────────────────────────────────────────
  async function spinLink() {
    const active = links.filter(l=>!isExpired(l)&&!l.flagged);
    if (!active.length) { showToast("No links available","alert-circle"); return; }

    const btn   = document.getElementById("spinBtn");
    const sm    = document.getElementById("slotMachine");
    const reel  = document.getElementById("slotReel");
    const chosen = active[Math.floor(Math.random()*active.length)];
    btn.disabled = true;

    const dummy = Array.from({length:20},(_,i)=>active[i%active.length].url);
    reel.innerHTML = dummy.map(u=>`<div class="slot-item">${u}</div>`).join("") +
      `<div class="slot-item" style="color:var(--edf-orange);font-weight:600;">${chosen.url}</div>`;

    sm.classList.remove("settling");
    sm.classList.add("spinning");
    setTimeout(()=>{
      sm.classList.remove("spinning");
      sm.classList.add("settling");
      setTimeout(()=>{
        reel.innerHTML=`<div class="slot-item" style="color:var(--edf-orange);font-weight:600;">${chosen.url}</div>`;
        sm.classList.remove("settling");
        btn.disabled=false;
      },450);
    },620);

    currentSpunUrl = chosen.url;
    currentSpunId  = chosen.id;
    document.getElementById("spinUrl").textContent  = chosen.url;
    document.getElementById("spinNick").textContent = chosen.nick ? `Submitted by: u/${chosen.nick}` : "";
    document.getElementById("spinResult").classList.add("show");

    try { await updateDoc(doc(db,"links",chosen.id),{ generated: increment(1) }); } catch(e){}
    bumpStat("generated");
  }

  async function recordClick() {
    if (!currentSpunId) return;
    try { await updateDoc(doc(db,"links",currentSpunId),{ clicks: increment(1) }); } catch(e){}
    bumpStat("clicked");
  }

  async function copySpun() {
    if (!currentSpunUrl) return;
    await navigator.clipboard.writeText(currentSpunUrl);
    showToast("Link copied to clipboard!","copy");
    recordClick();
  }

  function openSpun() {
    if (!currentSpunUrl) return;
    recordClick();
    window.open(currentSpunUrl,"_blank");
  }

  function shareSpun() {
    if (!currentSpunUrl) return;
    if (navigator.share) {
      navigator.share({ title:"EDF Energy referral link", text:"Use this EDF referral link:", url:currentSpunUrl })
        .then(()=>{ recordClick(); showToast("Shared!","share"); }).catch(()=>{});
    } else {
      navigator.clipboard.writeText(currentSpunUrl).then(()=>showToast("Link copied — paste to share!","copy"));
      recordClick();
    }
  }

  // ── Remove ───────────────────────────────────────────────────
  function promptRemove(id) {
    pendingRemoveId = id;
    document.getElementById("removePinInput").value = "";
    document.getElementById("removePinMsg").style.display = "none";
    document.getElementById("removeOverlay").classList.add("show");
  }

  async function confirmRemove() {
    const pin = document.getElementById("removePinInput").value.trim();
    const msg = document.getElementById("removePinMsg");
    if (!pin) { showMsg(msg,"Please enter your PIN.","#e74c3c"); return; }
    if (!pendingRemoveId) return;
    const linkData = links.find(l=>l.id===pendingRemoveId);
    if (!linkData) { showMsg(msg,"Link not found.","#e74c3c"); return; }
    if (linkData.pin !== pin) { showMsg(msg,"Incorrect PIN. Only the original submitter can remove this link.","#e74c3c"); return; }
    try {
      await deleteDoc(doc(db,"links",pendingRemoveId));
      closeModal("removeOverlay");
      showToast("Link removed","check");
    } catch(e) { showMsg(msg,"Error removing — please try again.","#e74c3c"); }
  }

  // ── Report ───────────────────────────────────────────────────
  function promptReport() {
    if (!currentSpunUrl) return;
    pendingReportId = currentSpunId;
    document.getElementById("reportReason").value = "";
    document.getElementById("reportMsg").style.display = "none";
    document.getElementById("reportOverlay").classList.add("show");
  }

  async function submitReport() {
    const reason = document.getElementById("reportReason").value.trim();
    const msg    = document.getElementById("reportMsg");
    if (!reason) { showMsg(msg,"Please describe the issue.","#e74c3c"); return; }
    if (!pendingReportId) return;
    try {
      await updateDoc(doc(db,"links",pendingReportId),{ flagged:true, flagReason:reason });
      closeModal("reportOverlay");
      showToast("Report submitted — an admin will review it","flag");
    } catch(e) { showMsg(msg,"Error submitting report — please try again.","#e74c3c"); }
  }

  // ── Admin ────────────────────────────────────────────────────
  function openAdminLogin() {
    document.getElementById("adminPassInput").value = "";
    document.getElementById("adminLoginMsg").style.display = "none";
    document.getElementById("adminLoginOverlay").classList.add("show");
  }

  function checkAdminPass() {
    if (document.getElementById("adminPassInput").value === ADMIN_PASS) {
      closeModal("adminLoginOverlay");
      openAdminPanel();
    } else {
      showMsg(document.getElementById("adminLoginMsg"),"Incorrect password.","#e74c3c");
    }
  }

  function adminTab(mode) {
    adminTabMode = mode;
    ["all","flagged","expired"].forEach(t=>{
      document.getElementById("tab"+t.charAt(0).toUpperCase()+t.slice(1))
        .classList.toggle("active", t===mode);
    });
    renderAdminList();
  }

  function openAdminPanel() {
    document.getElementById("adm-submitted").textContent = links.filter(l=>!isExpired(l)).length;
    document.getElementById("adm-generated").textContent = globalStats.generated||0;
    document.getElementById("adm-clicked").textContent   = globalStats.clicked||0;
    const flagCount = links.filter(l=>l.flagged).length;
    document.getElementById("flagCount").textContent = flagCount ? `(${flagCount})` : "";
    adminTabMode="all";
    ["all","flagged","expired"].forEach(t=>{
      document.getElementById("tab"+t.charAt(0).toUpperCase()+t.slice(1))
        .classList.toggle("active", t==="all");
    });
    renderAdminList();
    document.getElementById("adminPanelOverlay").classList.add("show");
  }

  function renderAdminList() {
    const el = document.getElementById("adminLinkList");
    let subset = links;
    if (adminTabMode==="flagged") subset=links.filter(l=>l.flagged);
    else if (adminTabMode==="expired") subset=links.filter(l=>isExpired(l));
    if (!subset.length) { el.innerHTML='<div class="empty-state" style="padding:1rem 0;"><i class="ti ti-link-off"></i>Nothing here.</div>'; return; }
    el.innerHTML = subset.map(l=>`
      <div class="adm-link-row">
        <div class="adm-link-url">${l.url}</div>
        ${l.flagReason ? `<div class="adm-flag-reason"><i class="ti ti-flag" style="font-size:11px;"></i> "${l.flagReason}"</div>` : ""}
        <div class="adm-link-meta">
          ${l.nick ? `<span style="font-size:11px;color:var(--clr-text-muted);">u/${l.nick}</span>` : ""}
          ${expiryBadge(l)}
          ${l.flagged ? `<span class="adm-flag"><i class="ti ti-flag" style="font-size:10px;"></i> Flagged</span>` : ""}
          <span style="font-size:11px;color:var(--clr-text-muted);"><i class="ti ti-refresh" style="font-size:11px;"></i> ${l.generated||0} gen</span>
          <span style="font-size:11px;color:var(--clr-text-muted);"><i class="ti ti-mouse" style="font-size:11px;"></i> ${l.clicks||0} clicks</span>
        </div>
        <div class="adm-actions">
          <button class="btn btn-danger btn-sm" onclick="window._adminRemove('${l.id}')"><i class="ti ti-trash"></i> Remove</button>
          ${l.flagged ? `<button class="btn btn-success btn-sm" onclick="window._adminClearFlag('${l.id}')"><i class="ti ti-check"></i> Clear flag</button>` : ""}
        </div>
      </div>`).join("");
  }

  async function adminRemove(id) {
    try { await deleteDoc(doc(db,"links",id)); openAdminPanel(); showToast("Link removed","check"); }
    catch(e) { showToast("Error removing link","alert-circle"); }
  }

  async function adminClearFlag(id) {
    try { await updateDoc(doc(db,"links",id),{flagged:false,flagReason:""}); openAdminPanel(); }
    catch(e) { showToast("Error clearing flag","alert-circle"); }
  }

  // ── Rate bar ─────────────────────────────────────────────────
  function updateRateBar() {
    const count = getRateCount();
    const pct   = Math.min(100, Math.round((count/MAX_DAILY)*100));
    document.getElementById("rateLimitBar").style.display   = "block";
    document.getElementById("rateLimitText").textContent    = `${count}/${MAX_DAILY} today`;
    const bar = document.getElementById("rateBarFill");
    bar.style.width      = pct+"%";
    bar.style.background = count>=MAX_DAILY ? "#e74c3c" : "var(--edf-orange)";
  }

  // ── Expose to HTML onclick handlers ─────────────────────────
  window._submitLink    = submitLink;
  window._spinLink      = spinLink;
  window._copySpun      = copySpun;
  window._openSpun      = openSpun;
  window._shareSpun     = shareSpun;
  window._promptRemove  = promptRemove;
  window._confirmRemove = confirmRemove;
  window._promptReport  = promptReport;
  window._submitReport  = submitReport;
  window._openAdminLogin= openAdminLogin;
  window._checkAdminPass= checkAdminPass;
  window._adminTab      = adminTab;
  window._adminRemove   = adminRemove;
  window._adminClearFlag= adminClearFlag;
  window._closeModal    = closeModal;

  // ── Keyboard shortcuts ───────────────────────────────────────
  document.addEventListener("DOMContentLoaded", ()=>{
    document.getElementById("adminPassInput").addEventListener("keydown",  e=>{if(e.key==="Enter")checkAdminPass();});
    document.getElementById("submitUrl").addEventListener("keydown",       e=>{if(e.key==="Enter")submitLink();});
    document.getElementById("removePinInput").addEventListener("keydown",  e=>{if(e.key==="Enter")confirmRemove();});
    updateRateBar();
  });

  // ── Init ─────────────────────────────────────────────────────
  await loadStats();
  listenLinks();
</script>

<style>
  :root {
    --edf-orange:      #FF6600;
    --edf-orange-pale: #FFF0E6;
    --edf-blue:        #003366;
    --edf-blue-mid:    #005299;
    --edf-green:       #00A651;
    --edf-green-pale:  #E6F7EE;
    --clr-bg:          #f4f5f7;
    --clr-card:        #ffffff;
    --clr-border:      #e0e4ea;
    --clr-text:        #1a1a2e;
    --clr-text-muted:  #6b7280;
    --clr-input-bg:    #ffffff;
    --clr-slot-bg:     #f0f2f5;
    --clr-result-bg:   #f0f2f5;
    --clr-result-border:#d1d5db;
    --radius-sm: 6px;
    --radius-md: 10px;
    --radius-lg: 14px;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    background: var(--clr-bg);
    color: var(--clr-text);
    min-height: 100vh;
    padding: 1.5rem 1rem 3rem;
  }

  .app { max-width: 660px; margin: 0 auto; }

  /* ── Header ── */
  .header {
    background: var(--edf-blue);
    color: white;
    padding: 1.25rem 1.5rem;
    border-radius: var(--radius-lg);
    margin-bottom: 1.5rem;
    display: flex; align-items: flex-start; justify-content: space-between; gap: 12px;
  }
  .header-left   { display: flex; align-items: flex-start; gap: 12px; }
  .edf-logo      { background: var(--edf-orange); color: white; font-weight: 700; font-size: 14px; padding: 6px 10px; border-radius: var(--radius-sm); letter-spacing: 0.5px; flex-shrink: 0; margin-top: 2px; }
  .h-title       { font-size: 17px; font-weight: 600; color: white; line-height: 1.3; }
  .h-sub         { font-size: 12px; color: rgba(255,255,255,0.72); margin-top: 4px; line-height: 1.5; }
  .h-sub a       { color: var(--edf-orange); text-decoration: underline; }
  .h-sub a:hover { opacity: 0.85; }
  .admin-btn     { background: rgba(255,255,255,0.12); border: 1px solid rgba(255,255,255,0.3); color: white; padding: 6px 12px; border-radius: var(--radius-md); cursor: pointer; font-size: 13px; display: flex; align-items: center; gap: 6px; flex-shrink: 0; white-space: nowrap; }
  .admin-btn:hover { background: rgba(255,255,255,0.22); }

  /* ── Stats ── */
  .stats-grid { display: grid; grid-template-columns: repeat(3,1fr); gap: 10px; margin-bottom: 1.25rem; }
  .stat-card  { background: var(--clr-card); border: 1px solid var(--clr-border); border-radius: var(--radius-md); padding: 14px 16px; text-align: center; }
  .stat-num   { font-size: 28px; font-weight: 700; color: var(--edf-orange); }
  .stat-label { font-size: 12px; color: var(--clr-text-muted); margin-top: 3px; }

  /* ── Cards ── */
  .card       { background: var(--clr-card); border: 1px solid var(--clr-border); border-radius: var(--radius-lg); padding: 1.25rem; margin-bottom: 1rem; }
  .card-title { font-size: 14px; font-weight: 600; color: var(--clr-text); margin-bottom: 1rem; display: flex; align-items: center; gap: 8px; }
  .card-title i { font-size: 18px; color: var(--edf-orange); }

  /* ── Inputs ── */
  .input-row          { display: flex; gap: 8px; }
  .input-row input    { flex: 1; }
  input[type=text],
  input[type=password],
  textarea {
    width: 100%; padding: 9px 12px;
    border: 1px solid var(--clr-border);
    border-radius: var(--radius-md);
    font-size: 14px;
    background: var(--clr-input-bg);
    color: var(--clr-text);
    font-family: inherit;
  }
  input:focus, textarea:focus { outline: none; border-color: var(--edf-orange); box-shadow: 0 0 0 3px rgba(255,102,0,0.12); }

  /* ── Buttons ── */
  .btn          { padding: 9px 16px; border-radius: var(--radius-md); border: none; cursor: pointer; font-size: 14px; font-weight: 600; display: inline-flex; align-items: center; gap: 6px; transition: opacity .15s, transform .1s; text-decoration: none; }
  .btn:hover    { opacity: 0.87; }
  .btn:active   { transform: scale(0.98); }
  .btn-primary  { background: var(--edf-orange); color: white; }
  .btn-blue     { background: var(--edf-blue); color: white; }
  .btn-success  { background: var(--edf-green); color: white; }
  .btn-ghost    { background: #f3f4f6; color: var(--clr-text); border: 1px solid var(--clr-border); }
  .btn-danger   { background: #e74c3c; color: white; }
  .btn-warn     { background: #e67e22; color: white; }
  .btn-sm       { padding: 6px 11px; font-size: 12px; }

  /* ── Slot machine ── */
  .slot-machine   { background: var(--clr-slot-bg); border: 1px solid var(--clr-border); border-radius: var(--radius-md); height: 40px; overflow: hidden; margin: 0 auto 12px; position: relative; }
  .slot-reel      { display: flex; flex-direction: column; align-items: center; }
  .slot-item      { font-size: 13px; font-family: monospace; color: var(--clr-text-muted); height: 40px; display: flex; align-items: center; padding: 0 12px; white-space: nowrap; overflow: hidden; max-width: 620px; text-overflow: ellipsis; }
  @keyframes reelSpin    { 0%{transform:translateY(0)} 100%{transform:translateY(-800px)} }
  @keyframes reelSettle  { 0%{transform:translateY(-800px)} 60%{transform:translateY(5px)} 80%{transform:translateY(-2px)} 100%{transform:translateY(0)} }
  .spinning .slot-reel  { animation: reelSpin 0.6s linear; }
  .settling .slot-reel  { animation: reelSettle 0.42s ease-out forwards; }
  .spin-section         { text-align: center; }

  /* ── Spin result ── */
  .spin-result          { background: var(--clr-result-bg); border: 1px solid var(--clr-result-border); border-radius: var(--radius-lg); padding: 1.25rem; margin-top: 1rem; display: none; }
  .spin-result.show     { display: block; }
  .spin-url             { font-size: 13px; font-family: monospace; color: var(--edf-orange); word-break: break-all; margin: 6px 0; font-weight: 600; }
  .spin-actions         { display: flex; gap: 8px; justify-content: center; margin-top: 12px; flex-wrap: wrap; }

  /* ── Link list ── */
  .list-item            { display: flex; align-items: center; gap: 10px; padding: 10px 12px; border: 1px solid var(--clr-border); border-radius: var(--radius-md); margin-bottom: 6px; }
  .list-item-url        { flex: 1; color: var(--edf-orange); overflow: hidden; text-overflow: ellipsis; white-space: nowrap; font-family: monospace; font-size: 12px; font-weight: 600; }

  /* ── Badges ── */
  .expiry-badge         { font-size: 10px; padding: 2px 7px; border-radius: 20px; white-space: nowrap; }
  .exp-ok               { background: var(--edf-green-pale); color: var(--edf-green); }
  .exp-soon             { background: #FFF3E0; color: #e67e22; }
  .exp-old              { background: #feeceb; color: #e74c3c; }
  .flagged-badge        { font-size: 10px; padding: 2px 7px; border-radius: 20px; background: #feeceb; color: #e74c3c; }

  /* ── Rate bar ── */
  .rate-bar-track       { height: 4px; border-radius: 2px; background: var(--clr-border); margin-top: 6px; }
  .rate-bar-fill        { height: 4px; border-radius: 2px; background: var(--edf-orange); transition: width .3s; }

  /* ── Info note ── */
  .info-note            { font-size: 12px; color: var(--clr-text-muted); background: #f9fafb; border: 1px solid var(--clr-border); border-radius: var(--radius-md); padding: 8px 12px; margin-top: 10px; display: flex; gap: 6px; align-items: flex-start; }

  /* ── Toast ── */
  .toast                { position: fixed; top: 14px; right: 14px; background: #1a1a2e; color: white; padding: 10px 16px; border-radius: var(--radius-md); font-size: 13px; z-index: 999; opacity: 0; pointer-events: none; display: flex; align-items: center; gap: 8px; transition: opacity .25s; box-shadow: 0 4px 16px rgba(0,0,0,0.2); }
  .toast.show           { opacity: 1; pointer-events: auto; }

  /* ── Overlays / Modals ── */
  .overlay              { position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 50; display: none; align-items: center; justify-content: center; padding: 1rem; }
  .overlay.show         { display: flex; }
  .modal                { background: var(--clr-card); border-radius: var(--radius-lg); padding: 1.5rem; width: 100%; max-width: 500px; border: 1px solid var(--clr-border); max-height: 90vh; overflow-y: auto; box-shadow: 0 8px 32px rgba(0,0,0,0.18); }
  .modal h2             { font-size: 16px; font-weight: 700; margin-bottom: 1rem; display: flex; align-items: center; gap: 8px; }
  .modal h2 i           { color: var(--edf-orange); }
  .modal-footer         { display: flex; gap: 8px; justify-content: flex-end; margin-top: 1rem; }

  /* ── Admin ── */
  .admin-stat-row       { display: grid; grid-template-columns: repeat(3,1fr); gap: 8px; margin-bottom: 1rem; }
  .admin-stat           { background: #eef2f8; border-radius: var(--radius-md); padding: 10px; text-align: center; }
  .admin-stat .n        { font-size: 22px; font-weight: 700; color: var(--edf-blue); }
  .admin-stat .l        { font-size: 11px; color: var(--edf-blue-mid); margin-top: 2px; }
  .tab-row              { display: flex; gap: 6px; margin-bottom: 1rem; flex-wrap: wrap; }
  .tab                  { padding: 6px 14px; border-radius: var(--radius-md); font-size: 13px; cursor: pointer; border: 1px solid var(--clr-border); background: #f3f4f6; color: var(--clr-text-muted); font-weight: 500; }
  .tab.active           { background: var(--edf-blue); color: white; border-color: var(--edf-blue); }
  .adm-link-row         { border: 1px solid var(--clr-border); border-radius: var(--radius-md); padding: 10px 12px; margin-bottom: 6px; }
  .adm-link-url         { font-family: monospace; font-size: 11px; color: var(--edf-orange); word-break: break-all; margin-bottom: 4px; font-weight: 600; }
  .adm-link-meta        { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
  .adm-flag             { font-size: 11px; padding: 2px 7px; border-radius: 10px; background: #feeceb; color: #e74c3c; }
  .adm-flag-reason      { font-size: 11px; color: #e74c3c; font-style: italic; margin-top: 3px; }
  .adm-actions          { display: flex; gap: 6px; margin-top: 8px; flex-wrap: wrap; }

  /* ── Empty state ── */
  .empty-state          { text-align: center; padding: 2rem 1rem; color: var(--clr-text-muted); font-size: 14px; }
  .empty-state i        { font-size: 38px; color: var(--clr-border); display: block; margin-bottom: 8px; }

  /* ── Report textarea ── */
  textarea              { resize: vertical; min-height: 72px; }
</style>
</head>
<body>
<div class="app">

  <!-- Header -->
  <div class="header">
    <div class="header-left">
      <div class="edf-logo">EDF</div>
      <div>
        <div class="h-title">r/EDFEnergy Referral Code Generator</div>
        <div class="h-sub">
          Share and Generate EDF Energy Referral Links &mdash; Brought to you by the reddit community
          <a href="https://www.reddit.com/r/EDFEnergy/" target="_blank" rel="noopener">r/EDFEnergy</a>
        </div>
      </div>
    </div>
    <button class="admin-btn" onclick="window._openAdminLogin()">
      <i class="ti ti-lock"></i> Admin
    </button>
  </div>

  <!-- Stats -->
  <div class="stats-grid">
    <div class="stat-card"><div class="stat-num" id="stat-submitted">…</div><div class="stat-label">Codes submitted</div></div>
    <div class="stat-card"><div class="stat-num" id="stat-generated">…</div><div class="stat-label">Times generated</div></div>
    <div class="stat-card"><div class="stat-num" id="stat-clicked">…</div><div class="stat-label">Links clicked</div></div>
  </div>

  <!-- Submit -->
  <div class="card">
    <div class="card-title"><i class="ti ti-link"></i> Submit your referral link</div>
    <div class="input-row">
      <input type="text" id="submitUrl" placeholder="https://refer.edfenergy.com/...">
      <button class="btn btn-primary" onclick="window._submitLink()">
        <i class="ti ti-plus"></i> Submit
      </button>
    </div>
    <input type="text" id="submitNick" placeholder="Your Reddit Username (Optional)" style="margin-top:8px;">
    <div id="submitMsg" style="margin-top:8px;font-size:13px;display:none;"></div>
    <div id="rateLimitBar" style="display:none;margin-top:10px;">
      <div style="display:flex;justify-content:space-between;font-size:11px;color:var(--clr-text-muted);">
        <span>Daily submission limit</span><span id="rateLimitText"></span>
      </div>
      <div class="rate-bar-track"><div class="rate-bar-fill" id="rateBarFill" style="width:0%;"></div></div>
    </div>
    <div class="info-note">
      <i class="ti ti-info-circle" style="font-size:14px;flex-shrink:0;margin-top:1px;"></i>
      Only official edfenergy.com referral links are accepted. Links expire automatically after 12 months.
    </div>
  </div>

  <!-- Spin -->
  <div class="card">
    <div class="card-title"><i class="ti ti-refresh"></i> Get a random referral link</div>
    <div class="spin-section">
      <div class="slot-machine" id="slotMachine">
        <div class="slot-reel" id="slotReel">
          <div class="slot-item" style="color:var(--clr-text-muted);">Press generate to spin a link</div>
        </div>
      </div>
      <button class="btn btn-blue" id="spinBtn" onclick="window._spinLink()" style="padding:10px 28px;font-size:15px;">
        <i class="ti ti-player-play"></i> Generate random link
      </button>
    </div>
    <div class="spin-result" id="spinResult">
      <div style="font-size:12px;color:var(--clr-text-muted);">Selected link</div>
      <div class="spin-url" id="spinUrl"></div>
      <div style="font-size:12px;color:var(--clr-text-muted);margin-top:2px;" id="spinNick"></div>
      <div class="spin-actions">
        <button class="btn btn-primary btn-sm" onclick="window._copySpun()"><i class="ti ti-copy"></i> Copy link</button>
        <button class="btn btn-primary btn-sm" onclick="window._openSpun()"><i class="ti ti-external-link"></i> Open link</button>
        <button class="btn btn-primary btn-sm" onclick="window._shareSpun()"><i class="ti ti-share"></i> Share</button>
        <button class="btn btn-warn btn-sm" onclick="window._promptReport()"><i class="ti ti-flag"></i> Report</button>
      </div>
    </div>
  </div>

  <!-- List -->
  <div class="card">
    <div class="card-title">
      <i class="ti ti-list"></i> Submitted links
      <span style="font-size:12px;font-weight:400;color:var(--clr-text-muted);margin-left:4px;">— use your PIN to remove your own</span>
    </div>
    <div id="linkList"><div class="empty-state"><i class="ti ti-loader"></i>Loading…</div></div>
  </div>

</div><!-- /.app -->

<!-- Remove modal -->
<div class="overlay" id="removeOverlay">
  <div class="modal">
    <h2><i class="ti ti-trash"></i> Remove your link</h2>
    <p style="font-size:14px;color:var(--clr-text-muted);margin-bottom:1rem;">Enter the 4-digit PIN you received when submitting to confirm removal.</p>
    <input type="password" id="removePinInput" placeholder="Your 4-digit PIN">
    <div id="removePinMsg" style="margin-top:6px;font-size:13px;display:none;"></div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="window._closeModal('removeOverlay')">Cancel</button>
      <button class="btn btn-danger" onclick="window._confirmRemove()"><i class="ti ti-trash"></i> Remove</button>
    </div>
  </div>
</div>

<!-- Report modal -->
<div class="overlay" id="reportOverlay">
  <div class="modal">
    <h2><i class="ti ti-flag"></i> Report this link</h2>
    <p style="font-size:14px;color:var(--clr-text-muted);margin-bottom:1rem;">Tell us why you're reporting this link. An admin will review it.</p>
    <textarea id="reportReason" placeholder="Describe the issue (e.g. broken link, spam, not an EDF link)…"></textarea>
    <div id="reportMsg" style="margin-top:6px;font-size:13px;display:none;"></div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="window._closeModal('reportOverlay')">Cancel</button>
      <button class="btn btn-warn" onclick="window._submitReport()"><i class="ti ti-flag"></i> Submit report</button>
    </div>
  </div>
</div>

<!-- Admin login modal -->
<div class="overlay" id="adminLoginOverlay">
  <div class="modal">
    <h2><i class="ti ti-lock"></i> Admin access</h2>
    <p style="font-size:14px;color:var(--clr-text-muted);margin-bottom:1rem;">Enter the admin password to manage submitted links.</p>
    <input type="password" id="adminPassInput" placeholder="Admin password">
    <div id="adminLoginMsg" style="margin-top:6px;font-size:13px;display:none;"></div>
    <div class="modal-footer">
      <button class="btn btn-ghost" onclick="window._closeModal('adminLoginOverlay')">Cancel</button>
      <button class="btn btn-primary" onclick="window._checkAdminPass()"><i class="ti ti-arrow-right"></i> Enter</button>
    </div>
  </div>
</div>

<!-- Admin panel modal -->
<div class="overlay" id="adminPanelOverlay">
  <div class="modal" style="max-width:580px;">
    <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:1rem;">
      <h2 style="margin:0;"><i class="ti ti-shield-check"></i> Admin panel</h2>
      <button class="btn btn-ghost btn-sm" onclick="window._closeModal('adminPanelOverlay')"><i class="ti ti-x"></i></button>
    </div>
    <div class="admin-stat-row">
      <div class="admin-stat"><div class="n" id="adm-submitted">0</div><div class="l">Active links</div></div>
      <div class="admin-stat"><div class="n" id="adm-generated">0</div><div class="l">Generated</div></div>
      <div class="admin-stat"><div class="n" id="adm-clicked">0</div><div class="l">Clicked</div></div>
    </div>
    <div class="tab-row">
      <div class="tab active" id="tabAll" onclick="window._adminTab('all')">All links</div>
      <div class="tab" id="tabFlagged" onclick="window._adminTab('flagged')">Flagged <span id="flagCount"></span></div>
      <div class="tab" id="tabExpired" onclick="window._adminTab('expired')">Expired</div>
    </div>
    <div id="adminLinkList"></div>
    <p style="font-size:11px;color:var(--clr-text-muted);margin-top:1rem;padding-top:1rem;border-top:1px solid var(--clr-border);">Change admin password via the <code>ADMIN_PASS</code> constant in the source.</p>
  </div>
</div>

<!-- Toast -->
<div class="toast" id="toast">
  <i class="ti ti-check" id="toastIcon"></i>
  <span id="toastMsg"></span>
</div>

</body>
</html>
