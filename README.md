<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>fluxe — админ</title>
<meta name="theme-color" content="#07140e">
<style>
  :root{ --bg:#07140e; --green:#00ff88; --text:#eaf9f1; --muted:#b6e3c8; }
  body{
    margin:0; color:var(--text);
    font-family:Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
    background:
      radial-gradient(1200px 800px at 10% -10%, #0b2a1c 0%, var(--bg) 40%),
      radial-gradient(1000px 700px at 110% 10%, #0c3a25 0%, var(--bg) 30%),
      var(--bg);
  }
  #bg-snow{ position:fixed; inset:0; width:100vw; height:100vh; z-index:0; pointer-events:none; display:block; }

  .wrap{ position:relative; z-index:1; max-width:1100px; margin:40px auto; padding:0 16px; }
  .box{ background: rgba(5,18,12,.7); border:1px solid rgba(0,255,136,.2); border-radius:14px; padding:16px; box-shadow: 0 8px 24px rgba(0,0,0,.35); }
  h1{ margin:0 0 16px; font-size:28px; color:#bfffe1; text-shadow:0 0 12px rgba(0,255,136,.25); }
  h2{ margin:14px 0 10px; font-size:18px; color:#bfffe1; }
  label{ display:block; margin-bottom:6px; color:#cfeede; font-size:14px; }
  input[type="text"], input[type="password"]{
    width:100%; padding:10px 12px; border-radius:10px; border:1px solid rgba(0,255,136,.2); background:#0f2219; color:#eaf9f1; outline:none;
  }
  select{
    width:180px; padding:8px; border-radius:10px; border:1px solid rgba(0,255,136,.3);
    background:#0f2219; color:#eaf9f1; outline:none; appearance:auto;
  }
  select option{ background:#0f2219; color:#eaf9f1; }
  .btn{ display:inline-flex; align-items:center; gap:8px; height:40px; padding:0 14px; border-radius:10px; background: linear-gradient(180deg, rgba(0,255,136,.95), rgba(0,230,120,.9)); color:#042d16; font-weight:800; border:none; cursor:pointer; }
  .btn.secondary{ background: transparent; color:#bfffe1; border:1px solid rgba(0,255,136,.25); }

  .row{ display:grid; grid-template-columns: 1fr auto; gap:10px; align-items:end; }
  #search{ width:min(320px, 100%); } /* короче первая строка поиска */

  .list{ display:grid; gap:10px; }
  .item{ display:grid; gap:8px; padding:12px; border-radius:12px; border:1px solid rgba(0,255,136,.18); background: rgba(5,18,12,.6); }
  .item .meta{ display:grid; grid-template-columns: 1fr 1fr; gap:6px; font-size:13px; color:#cfeede; }
  .actions{ display:flex; gap:8px; flex-wrap:wrap; }
  .status{ font-size:12px; padding:2px 8px; border-radius:999px; border:1px solid rgba(0,255,136,.25); color:#bfffe1; background: rgba(0,255,136,.08); }
  .muted{ color:#8acfb1; font-size:13px; }
  .topbar{ display:flex; justify-content:space-between; align-items:center; margin-bottom:16px; gap:12px; }
  .right{ display:flex; align-items:center; gap:10px; }
  .avatar{
    width:40px; height:40px; border-radius:50%; overflow:hidden; border:1px solid rgba(0,255,136,.3);
    box-shadow: 0 0 10px rgba(0,255,136,.3); background: rgba(0,255,136,.1);
  }
  .avatar img{ width:100%; height:100%; object-fit:cover; display:block; }
  .hidden{ display:none !important; }
  .reveal{ opacity:0; transform: translateY(10px); transition: opacity .5s ease, transform .5s ease; }
  .reveal.in{ opacity:1; transform:none; }

  /* Табы */
  .tabs{ display:flex; gap:8px; margin-bottom:16px; }
  .tab{ padding:8px 12px; border-radius:10px; border:1px solid rgba(0,255,136,.25); background: rgba(0,255,136,.08); color:#bfffe1; cursor:pointer; font-weight:700; }
  .tab.active{ background: linear-gradient(180deg, rgba(0,255,136,.2), rgba(0,255,136,.12)); border-color: rgba(0,255,136,.45); }

  @media (max-width:640px){
    .row{ grid-template-columns:1fr }
    .item .meta{ grid-template-columns:1fr }
    select{ width:100% }
    #search{ width:100% }
  }
</style>
</head>
<body>
<canvas id="bg-snow" aria-hidden="true"></canvas>

<div class="wrap">
  <div class="box reveal" id="loginBox">
    <div class="topbar">
      <h1>Вход в админ-панель</h1>
      <div class="right">
        <div class="avatar" title="Аватар админа">
          <img id="avatarImg" alt="avatar" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='80' height='80'><rect width='100%' height='100%' fill='%230a1f17'/><text x='50%' y='55%' font-size='34' fill='%2300ff88' text-anchor='middle' font-family='Arial'>&#128272;</text></svg>">
        </div>
        <input type="file" id="avatarInput" accept="image/*" class="hidden">
        <button class="btn secondary" id="changeAvatar">Сменить аватар</button>
      </div>
    </div>
    <label for="login">Логин</label>
    <input type="text" id="login" placeholder="support_pelemeuu">
    <label for="pass" style="margin-top:10px;">Пароль</label>
    <input type="password" id="pass" placeholder="Пароль">
    <div style="margin-top:12px; display:flex; gap:8px;">
      <button class="btn" id="doLogin">Войти</button>
      <span class="muted">Логин: support_pelemeuu</span>
    </div>
  </div>

  <!-- Табы после входа -->
  <div class="box hidden reveal" id="tabsBox">
    <div class="tabs">
      <button class="tab active" id="tabApps">Заявки</button>
      <button class="tab" id="tabHistory">История</button>
    </div>
  </div>

  <div class="box hidden reveal" id="appBox">
    <div class="topbar">
      <h1>Заявки</h1>
      <div class="right">
        <div class="avatar" title="Аватар админа">
          <img id="avatarImg2" alt="avatar">
        </div>
        <input type="file" id="avatarInput2" accept="image/*" class="hidden">
        <button class="btn secondary" id="changeAvatar2">Сменить аватар</button>
        <button class="btn secondary" id="logout">Выйти</button>
        <button class="btn secondary" id="clearAll">Очистить заявки</button>
      </div>
    </div>

    <div class="row">
      <div>
        <label>Поиск</label>
        <input type="text" id="search" placeholder="Поиск по нику или названию">
      </div>
      <div>
        <label>Фильтр</label>
        <select id="filter">
          <option value="all">Все</option>
          <option value="pending">Ожидают</option>
          <option value="approved">Одобрены</option>
          <option value="rejected">Отклонены</option>
        </select>
      </div>
    </div>

    <h2 style="margin-top:16px;">Список заявок</h2>
    <div class="list" id="list"></div>
    <div class="muted" id="empty">Пока пусто.</div>
  </div>

  <div class="box hidden reveal" id="historyBox">
    <div class="topbar">
      <h1>История действий</h1>
      <div class="right">
        <input type="text" id="histSearch" placeholder="Поиск в истории" style="padding:8px 10px; border-radius:10px; border:1px solid rgba(0,255,136,.2); background:#0f2219; color:#eaf9f1;">
        <!-- Кнопка очистки истории удалена -->
      </div>
    </div>
    <div class="list" id="histList"></div>
    <div class="muted" id="histEmpty">История пуста.</div>
  </div>
</div>

<script>
  // Анимации появления
  const animated = document.querySelectorAll('.reveal');
  const io = new IntersectionObserver((entries, obs) => {
    entries.forEach(e => { if(e.isIntersecting){ e.target.classList.add('in'); obs.unobserve(e.target); } });
  }, { threshold: .1 }); animated.forEach(el => io.observe(el));

  // Снег (фон)
  (function snow(){
    const prefersReduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
    if(prefersReduce) return;
    const c = document.getElementById('bg-snow');
    const ctx = c.getContext('2d'); let flakes=[], raf=null;
    function setCanvasSize(){
      const dpr = Math.max(1, window.devicePixelRatio || 1);
      const w = Math.max(document.documentElement.clientWidth, window.innerWidth || 0);
      const h = Math.max(document.documentElement.clientHeight, window.innerHeight || 0);
      c.width = Math.floor(w*dpr); c.height = Math.floor(h*dpr);
      c.style.width = w+'px'; c.style.height = h+'px';
      ctx.setTransform(1,0,0,1,0,0); ctx.scale(dpr,dpr);
      adjustFlakes(w,h);
    }
    function makeFlake(w,h){ return { x:Math.random()*w, y:Math.random()*h, r:1+Math.random()*2.2, s:.4+Math.random()*1.2, d:Math.random()*1.5 }; }
    function adjustFlakes(w,h){
      const target = Math.floor(w*h*0.00012);
      if(flakes.length===0){ flakes = new Array(target).fill(0).map(()=>makeFlake(w,h)); return; }
      if(flakes.length<target){ for(let i=flakes.length;i<target;i++) flakes.push(makeFlake(w,h)); }
      else if(flakes.length>target){ flakes.length = target; }
    }
    function step(){
      const w=c.clientWidth,h=c.clientHeight;
      ctx.clearRect(0,0,w,h);
      ctx.fillStyle='rgba(255,255,255,.9)';
      for(const f of flakes){
        f.y+=f.s; f.x += Math.sin((f.y+f.d)*0.01)*0.6;
        if(f.y>h+5){ f.y=-10; f.x=Math.random()*w; }
        ctx.beginPath(); ctx.arc(f.x,f.y,f.r,0,Math.PI*2); ctx.fill();
      }
      raf=requestAnimationFrame(step);
    }
    function start(){ if(raf) return; setCanvasSize(); raf=requestAnimationFrame(step); }
    function stop(){ if(raf){ cancelAnimationFrame(raf); raf=null; } }
    window.addEventListener('resize', setCanvasSize);
    document.addEventListener('visibilitychange', ()=>{ if(document.hidden) stop(); else start(); });
    start();
  })();

  // Авторизация
  const CRED = {
    login: 'support_pelemeuu',
    pass: '9358gj37&HIUYegs8u84hgY&Hugr87w4hudgj%UgnjdshgYUghsj4njefguydhGUSRIHGUSDH47wyudgf'
  };
  const loginBox = document.getElementById('loginBox');
  const tabsBox = document.getElementById('tabsBox');
  const appBox = document.getElementById('appBox');
  const historyBox = document.getElementById('historyBox');
  const doLogin = document.getElementById('doLogin');
  const logout = document.getElementById('logout');

  // Табы
  const tabApps = document.getElementById('tabApps');
  const tabHistory = document.getElementById('tabHistory');
  function showTab(name){
    if(name==='apps'){
      appBox.classList.remove('hidden'); historyBox.classList.add('hidden');
      tabApps.classList.add('active'); tabHistory.classList.remove('active');
    }else{
      appBox.classList.add('hidden'); historyBox.classList.remove('hidden');
      tabHistory.classList.add('active'); tabApps.classList.remove('active');
    }
  }
  tabApps.addEventListener('click', ()=>showTab('apps'));
  tabHistory.addEventListener('click', ()=>showTab('history'));

  // Аватар
  const avatarImg = document.getElementById('avatarImg');
  const avatarImg2 = document.getElementById('avatarImg2');
  const avatarInput = document.getElementById('avatarInput');
  const avatarInput2 = document.getElementById('avatarInput2');
  const changeAvatar = document.getElementById('changeAvatar');
  const changeAvatar2 = document.getElementById('changeAvatar2');

  function loadAvatar(){
    const src = localStorage.getItem('fluxeAdminAvatar');
    if(src){ avatarImg.src = src; avatarImg2.src = src; }
    else { avatarImg2.src = avatarImg.src; }
  }
  function chooseAvatar(input){
    input.click();
    input.onchange = async () => {
      const file = input.files && input.files[0];
      if(!file) return;
      const dataUrl = await fileToDataURL(file);
      const resized = await resizeImage(dataUrl, 128);
      localStorage.setItem('fluxeAdminAvatar', resized);
      loadAvatar();
    };
  }
  function fileToDataURL(file){ return new Promise(res=>{ const fr = new FileReader(); fr.onload=()=>res(fr.result); fr.readAsDataURL(file); }); }
  function resizeImage(src, size){ return new Promise(res=>{ const img=new Image(); img.onload=()=>{ const c=document.createElement('canvas'); c.width=size; c.height=size; const ctx=c.getContext('2d'); ctx.drawImage(img,0,0,size,size); res(c.toDataURL('image/webp',0.9)); }; img.src=src; }); }
  changeAvatar.addEventListener('click', ()=>chooseAvatar(avatarInput));
  changeAvatar2.addEventListener('click', ()=>chooseAvatar(avatarInput2));
  loadAvatar();

  // База заявок
  const listEl = document.getElementById('list');
  const emptyEl = document.getElementById('empty');
  const clearBtn = document.getElementById('clearAll');
  const searchEl = document.getElementById('search');
  const filterEl = document.getElementById('filter');

  function getDb(){
    try{ return JSON.parse(localStorage.getItem('fluxeApplications')) || []; }
    catch(e){ return []; }
  }
  function setDb(data){ localStorage.setItem('fluxeApplications', JSON.stringify(data)); }
  function addApp(app){ const db=getDb(); db.unshift(app); setDb(db); render(); }
  function updateStatus(id, status, reason){
    const db = getDb().map(a => a.id===id ? {...a, status, reason: status==='rejected' ? (reason||'') : a.reason} : a);
    setDb(db);
    const app = db.find(a=>a.id===id);
    if(status==='approved') addLog('approved', app);
    if(status==='rejected') addLog('rejected', app, reason||'');
    render();
  }
  function removeApp(id){
    const db = getDb();
    const app = db.find(a=>a.id===id);
    const rest = db.filter(a => a.id!==id);
    setDb(rest);
    if(app) addLog('removed', app);
    render();
  }

  // История
  const histList = document.getElementById('histList');
  const histEmpty = document.getElementById('histEmpty');
  const histSearch = document.getElementById('histSearch');

  function getHistory(){
    try{ return JSON.parse(localStorage.getItem('fluxeHistory')) || []; }
    catch(e){ return []; }
  }
  function setHistory(arr){ localStorage.setItem('fluxeHistory', JSON.stringify(arr)); }
  function addLog(action, app, reason){
    const rec = {
      id: 'log_'+Date.now()+'_'+Math.random().toString(36).slice(2,8),
      action, appId: app.id, title: app.title, nick: app.nick,
      reason: reason || '', ts: Date.now()
    };
    const log = getHistory(); log.unshift(rec); setHistory(log); renderHistory();
  }
  function renderHistory(){
    const q = (histSearch.value||'').toLowerCase().trim();
    const log = getHistory().filter(l => {
      const s = [l.action, l.title, l.nick, l.reason].join(' ').toLowerCase();
      return s.includes(q);
    });
    histList.innerHTML = '';
    if(log.length===0){ histEmpty.style.display='block'; return; }
    histEmpty.style.display='none';
    for(const l of log){
      const div = document.createElement('div');
      div.className = 'item';
      const actionText = l.action==='submitted'?'Создана': l.action==='approved'?'Одобрена': l.action==='rejected'?'Отклонена':'Удалена';
      div.innerHTML = `
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <strong>${escapeHtml(l.title)}</strong>
          <span class="status">${actionText}</span>
        </div>
        <div class="meta">
          <div><b>Ник:</b> ${escapeHtml(l.nick)}</div>
          <div><b>Время:</b> ${new Date(l.ts).toLocaleString()}</div>
          ${l.reason ? `<div style="grid-column:1/-1;"><b>Причина:</b> ${escapeHtml(l.reason)}</div>` : ''}
        </div>
      `;
      histList.appendChild(div);
    }
  }
  histSearch.addEventListener('input', renderHistory);

  // Рендер заявок
  function render(){
    const q = (searchEl.value||'').toLowerCase().trim();
    const f = filterEl.value;
    const data = getDb().filter(a => {
      const matches = a.nick.toLowerCase().includes(q) || a.title.toLowerCase().includes(q);
      const statusOk = f==='all' ? true : a.status===f;
      return matches && statusOk;
    });

    listEl.innerHTML = '';
    if(data.length===0){ emptyEl.style.display='block'; return; }
    emptyEl.style.display='none';

    data.forEach(a => {
      const item = document.createElement('div');
      item.className = 'item';
      const statusText = a.status==='pending'?'Ожидает': a.status==='approved'?'Одобрена':'Отклонена';
      item.innerHTML = `
        <div style="display:flex; justify-content:space-between; align-items:center;">
          <strong>${escapeHtml(a.title)}</strong>
          <span class="status">${statusText}</span>
        </div>
        <div class="meta">
          <div><b>Ник:</b> ${escapeHtml(a.nick)}</div>
          <div><b>Время:</b> ${new Date(a.ts).toLocaleString()}</div>
          <div style="grid-column:1 / -1; word-break:break-all;"><b>VirusTotal:</b> <a href="${a.virustotal}" target="_blank" rel="noopener noreferrer">${a.virustotal}</a></div>
          <div style="grid-column:1 / -1; word-break:break-all;"><b>Ссылка:</b> <a href="${a.link}" target="_blank" rel="noopener noreferrer">${a.link}</a></div>
          ${a.status==='rejected' && a.reason ? `<div style="grid-column:1/-1;"><b>Причина отклонения:</b> ${escapeHtml(a.reason)}</div>` : ''}
        </div>
        <div class="actions">
          <button class="btn" data-act="approve">Одобрить</button>
          <button class="btn secondary" data-act="reject">Отклонить</button>
          <button class="btn secondary" data-act="remove">Удалить</button>
        </div>
      `;
      item.querySelector('[data-act="approve"]').addEventListener('click', () => updateStatus(a.id,'approved'));
      item.querySelector('[data-act="reject"]').addEventListener('click', () => {
        const reason = prompt('Укажите причину отклонения:','');
        if(reason===null) return; // отмена
        updateStatus(a.id,'rejected', reason.trim());
      });
      item.querySelector('[data-act="remove"]').addEventListener('click', () => {
        if(confirm('Удалить заявку?')) removeApp(a.id);
      });
      listEl.appendChild(item);
    });
  }

  function escapeHtml(s){ return String(s||'').replace(/[&<>"']/g, m => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[m])); }

  // Логирование отправленных заявок из submit.html
  function syncSubmittedLogs(){
    const apps = getDb();
    const known = new Set((JSON.parse(localStorage.getItem('fluxeKnownIds')) || []));
    const newOnes = apps.filter(a => !known.has(a.id));
    if(newOnes.length){
      newOnes.forEach(a => addLog('submitted', a));
      const updated = [...known, ...newOnes.map(a=>a.id)];
      localStorage.setItem('fluxeKnownIds', JSON.stringify(updated));
    }
  }
  window.addEventListener('storage', (e) => {
    if(e.key === 'fluxeApplications'){ syncSubmittedLogs(); render(); }
  });

  // Очистить все заявки
  clearBtn.addEventListener('click', () => {
    if(confirm('Очистить все заявки?')){
      localStorage.removeItem('fluxeApplications');
      localStorage.removeItem('fluxeKnownIds');
      render();
    }
  });

  // Поиск/фильтр
  searchEl.addEventListener('input', render);
  filterEl.addEventListener('change', render);

  // Логин/логаут
  doLogin.addEventListener('click', login);
  function login(){
    const user = document.getElementById('login').value.trim();
    const pass = document.getElementById('pass').value;
    if(user === CRED.login && pass === CRED.pass){
      localStorage.setItem('fluxeAdminToken','ok');
      loginBox.classList.add('hidden');
      tabsBox.classList.remove('hidden');
      appBox.classList.remove('hidden');
      historyBox.classList.add('hidden');
      syncSubmittedLogs();
      render();
      renderHistory();
    }else{
      alert('Неверный логин или пароль');
    }
  }
  logout.addEventListener('click', () => {
    localStorage.removeItem('fluxeAdminToken');
    tabsBox.classList.add('hidden');
    loginBox.classList.remove('hidden');
    appBox.classList.add('hidden');
    historyBox.classList.add('hidden');
  });

  // Автовход
  if(localStorage.getItem('fluxeAdminToken')==='ok'){
    loginBox.classList.add('hidden');
    tabsBox.classList.remove('hidden');
    appBox.classList.remove('hidden');
    historyBox.classList.add('hidden');
    syncSubmittedLogs();
    render();
    renderHistory();
  }
</script>
</body>
</html>
