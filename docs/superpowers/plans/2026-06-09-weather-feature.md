# Weather Feature — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add real-time Hong Kong weather info to the hk-transit app using the HKO Open Data API.

**Architecture:** Single-file HTML app (`src/index.html`). Add ~200 lines JS + ~100 lines CSS for weather. Uses `STATE_WEATHER` object for caching and auto-refresh every 10 min. Three display surfaces: header weather strip (always visible), home page weather card, and a slide-up detail panel.

**Tech Stack:** Vanilla JS, HKO Open Data API (`data.weather.gov.hk/weatherAPI/opendata/weather.php`), existing CSS variables and patterns.

**Files modified:** `src/index.html` (the only source file)

---

### Task 1: Add Weather CSS

**Files:**
- Modify: `src/index.html` (insert new CSS block after existing traffic styles, before responsive section line ~1249)

**Add ~110 lines of CSS** for three components:

- [ ] **Step 1: Insert weather CSS block**

Find the line `/* ========================================================================
   8. RESPONSIVE TWEAKS
   ======================================================================== */` (~line 1249).

Insert BEFORE that block:

```css
/* ========================================================================
   8. WEATHER
   ======================================================================== */
/* Header weather strip */
.weather-header{
  display:flex;align-items:center;gap:6px;
  padding:6px 10px;border-radius:8px;
  background:rgba(255,255,255,0.04);border:1px solid var(--border);
  cursor:pointer;
  transition:background .2s var(--ease-out),border-color .2s var(--ease-out);
  -webkit-tap-highlight-color:transparent;
  position:relative;
  flex-shrink:0;
}
.weather-header:hover{background:rgba(255,255,255,0.08);border-color:var(--border-strong)}
.weather-header:active{transform:scale(0.96)}
.weather-header .wt-temp{
  font-family:var(--font-mono);font-variant-numeric:tabular-nums;
  font-size:15px;font-weight:700;color:var(--neon-cyan);
  line-height:1;
}
.weather-header .wt-icon{font-size:15px;line-height:1;flex-shrink:0}
.weather-header .wt-humi{
  font-family:var(--font-mono);font-size:10px;color:var(--ink-2);
  display:flex;align-items:center;gap:2px;
}
.weather-header .wt-humi svg{width:12px;height:12px;stroke:currentColor;fill:none;stroke-width:2}
.weather-header.loading{opacity:.5;pointer-events:none}
.weather-header.error{display:none}

/* Home weather card */
.weather-card{
  margin:0 18px 18px;padding:18px 20px;
  border-radius:18px;
  background:linear-gradient(135deg,rgba(34,211,238,0.06),rgba(139,92,246,0.04));
  border:1px solid rgba(34,211,238,0.15);
  position:relative;overflow:hidden;
  transition:opacity .3s;
}
.weather-card::before{
  content:"";position:absolute;top:0;left:0;right:0;height:2px;
  background:linear-gradient(90deg,var(--neon-cyan),var(--neon-violet),transparent);
}
.weather-card .wc-head{
  display:flex;align-items:center;justify-content:space-between;gap:12px;
  margin-bottom:10px;
}
.weather-card .wc-label{
  font-family:var(--font-mono);font-size:10px;font-weight:600;
  color:var(--neon-cyan);text-transform:uppercase;letter-spacing:0.16em;
  display:flex;align-items:center;gap:6px;
}
.weather-card .wc-updated{
  font-family:var(--font-mono);font-size:9px;color:var(--ink-3);
}
.weather-card .wc-main{
  display:flex;align-items:center;gap:16px;
}
.weather-card .wc-temp{
  font-family:var(--font-mono);font-variant-numeric:tabular-nums;
  font-size:48px;font-weight:800;line-height:1;
  color:var(--ink-0);
  letter-spacing:-0.04em;
}
.weather-card .wc-temp .deg{
  font-size:24px;color:var(--neon-cyan);vertical-align:super;
}
.weather-card .wc-details{
  display:flex;flex-direction:column;gap:4px;min-width:0;
}
.weather-card .wc-row{
  display:flex;align-items:center;gap:8px;
  font-family:var(--font-mono);font-size:11px;color:var(--ink-1);
}
.weather-card .wc-row .val{
  color:var(--ink-0);font-weight:600;
}
.weather-card .wc-row svg{width:14px;height:14px;flex-shrink:0;color:var(--ink-2)}
.weather-card .wc-warnings{
  margin-top:10px;display:flex;flex-direction:column;gap:6px;
}
.weather-card .wc-warn{
  padding:8px 12px;border-radius:10px;
  display:flex;align-items:center;gap:8px;
  font-family:var(--font-zh);font-size:12px;font-weight:600;
  line-height:1.3;
}
.weather-card .wc-warn.amber{
  background:rgba(251,191,36,0.1);border:1px solid rgba(251,191,36,0.25);
  color:var(--neon-amber);
}
.weather-card .wc-warn.red{
  background:rgba(239,68,68,0.1);border:1px solid rgba(239,68,68,0.25);
  color:var(--neon-red);
}
.weather-card .wc-warn svg{width:16px;height:16px;flex-shrink:0}
.weather-card .wc-forecast{
  margin-top:10px;padding-top:10px;
  border-top:1px solid var(--border);
  font-family:var(--font-zh);font-size:12px;color:var(--ink-1);
  line-height:1.5;
}
.weather-card.loading .wc-temp,.weather-card.loading .wc-details{opacity:.3}
.weather-card.error{display:none}

/* Weather detail panel (slide-up, reuses map-sheet pattern) */
.weather-panel{
  position:fixed;left:0;right:0;bottom:0;z-index:50;
  background:linear-gradient(180deg,rgba(20,20,32,0.98),rgba(12,12,24,1));
  border-top:1px solid var(--border-strong);
  border-radius:24px 24px 0 0;
  padding:14px 20px calc(env(safe-area-inset-bottom,0) + 16px);
  transform:translateY(105%);transition:transform .4s cubic-bezier(.16,1,.3,1);
  box-shadow:0 -8px 40px rgba(0,0,0,0.6);
  max-height:70vh;overflow-y:auto;
  max-width:520px;margin:0 auto;
}
.weather-panel.show{transform:translateY(0)}
.weather-panel .wp-handle{
  width:36px;height:4px;border-radius:2px;background:var(--border-strong);
  margin:0 auto 16px;
}
.weather-panel .wp-title{
  font-family:var(--font-zh);font-weight:800;font-size:18px;
  margin-bottom:2px;
}
.weather-panel .wp-subtitle{
  font-family:var(--font-mono);font-size:10px;color:var(--ink-2);
  text-transform:uppercase;letter-spacing:0.14em;
  margin-bottom:16px;
}
.weather-panel .wp-grid{
  display:grid;grid-template-columns:1fr 1fr;gap:10px;
  margin-bottom:16px;
}
.weather-panel .wp-stat{
  padding:14px;border-radius:14px;
  background:rgba(255,255,255,0.03);border:1px solid var(--border);
}
.weather-panel .wp-stat .lbl{
  font-family:var(--font-mono);font-size:9px;color:var(--ink-2);
  text-transform:uppercase;letter-spacing:0.12em;margin-bottom:4px;
}
.weather-panel .wp-stat .val{
  font-family:var(--font-mono);font-variant-numeric:tabular-nums;
  font-size:22px;font-weight:700;color:var(--ink-0);
  line-height:1;
}
.weather-panel .wp-stat .val .unit{
  font-size:12px;font-weight:500;color:var(--ink-2);margin-left:2px;
}
.weather-panel .wp-warning{
  padding:10px 14px;border-radius:12px;
  display:flex;align-items:center;gap:10px;
  font-family:var(--font-zh);font-size:13px;font-weight:600;
  margin-bottom:10px;
}
.weather-panel .wp-warning.amber{
  background:rgba(251,191,36,0.1);border:1px solid rgba(251,191,36,0.2);
  color:var(--neon-amber);
}
.weather-panel .wp-warning.red{
  background:rgba(239,68,68,0.1);border:1px solid rgba(239,68,68,0.2);
  color:var(--neon-red);
}
.weather-panel .wp-forecast{
  padding:14px;border-radius:14px;
  background:rgba(34,211,238,0.04);border:1px solid rgba(34,211,238,0.12);
  font-family:var(--font-zh);font-size:13px;color:var(--ink-1);
  line-height:1.6;
}
/* Light theme overrides */
:root[data-theme="light"] .weather-card{background:linear-gradient(135deg,rgba(8,145,178,0.04),rgba(124,58,237,0.03));border-color:rgba(8,145,178,0.15)}
:root[data-theme="light"] .weather-panel{background:linear-gradient(180deg,rgba(255,255,255,0.98),rgba(248,249,253,1))}
:root[data-theme="light"] .weather-panel .wp-stat{background:rgba(13,17,28,0.03);border-color:rgba(13,17,28,0.10)}
:root[data-theme="light"] .weather-panel .wp-forecast{background:rgba(8,145,178,0.04);border-color:rgba(8,145,178,0.12)}
```

- [ ] **Step 2: Update the section numbering**

Change the responsive section comment from `8. RESPONSIVE TWEAKS` to `9. RESPONSIVE TWEAKS` (since we inserted weather as section 8).

```
/* ========================================================================
   9. RESPONSIVE TWEAKS
   ======================================================================== */
```

---

### Task 2: Add STATE_WEATHER + HKO Data Fetching

**Files:**
- Modify: `src/index.html` — Insert new state object + fetch functions right before the `/* ======================================================================== INIT ======================================================================== */` section (before line 3656)

- [ ] **Step 1: Add STATE_WEATHER after STATE_TRAFFIC**

Insert after `$('#trafficRefresh').addEventListener('click', loadTrafficNotices);` (after line 3654), before the `/* INIT */` section:

```js
/* ========================================================================
   WEATHER — HKO Open Data API
   ======================================================================== */
const WEATHER_URL = 'https://data.weather.gov.hk/weatherAPI/opendata/weather.php';
const STATE_WEATHER = {
  current: null,    // { temp, humi, icon, uv, rainfall, rainstorm, tcInfo, updateTime }
  forecast: null,   // { desc, icon, outlook }
  lastFetch: 0,
  interval: null,
  loading: false,
  error: null
};
const HKO_ICON_MAP = {
  '50': '☀️','51': '🌤️','52': '⛅','53': '☁️','54': '🌥️',
  '60': '🌧️','61': '🌦️','62': '🌧️','63': '🌧️','64': '🌧️',
  '65': '🌧️','70': '🌦️','71': '🌦️','72': '🌦️','73': '🌦️',
  '74': '🌧️','75': '🌧️','76': '🌧️','77': '🌧️','80': '🌦️',
  '81': '🌧️','82': '🌧️','83': '🌧️','84': '🌧️','85': '🌧️',
  '90': '⛈️','91': '⛈️','92': '⛈️','93': '⛈️','94': '⛈️',
};
const HKO_ICON_DEFAULT = '🌤️';

function hkoIcon(code){
  return HKO_ICON_MAP[String(code)] || HKO_ICON_DEFAULT;
}

async function fetchWeatherData(){
  if(STATE_WEATHER.loading) return;
  STATE_WEATHER.loading = true;
  const header = $('.weather-header');
  if(header) header.classList.add('loading');
  try{
    const [rhrread, flw] = await Promise.allSettled([
      fetchJSON(`${WEATHER_URL}?dataType=rhrread&lang=tc`, 8000, 1),
      fetchJSON(`${WEATHER_URL}?dataType=flw&lang=tc`, 8000, 1)
    ]);
    if(rhrread.status === 'fulfilled' && rhrread.value){
      const d = rhrread.value;
      // Get HKO main station temperature
      const hkoStation = (d.temperature?.data || []).find(s => s.place === '香港天文台')
        || (d.temperature?.data || [])[0];
      // Get humidity
      const humiData = (d.humidity?.data || [])[0];
      // Get UV
      const uvData = (d.uvindex?.data || [])[0];
      // Get rainfall
      const rainData = d.rainfall?.data?.[0];
      STATE_WEATHER.current = {
        temp: hkoStation?.value != null ? Math.round(hkoStation.value) : null,
        humi: humiData?.value != null ? humiData.value : null,
        icon: d.icon != null ? hkoIcon(d.icon) : null,
        iconCode: d.icon,
        uv: uvData?.value != null ? uvData.value : null,
        uvDesc: uvData?.desc || null,
        rainfall: rainData?.max || null,
        rainMsg: rainData?.msg || null,
        rainstorm: d.rainstorm || null,
        tcInfo: d.tcInfo || null,
        updateTime: d.updateTime || new Date().toISOString()
      };
      STATE_WEATHER.error = null;
    }
    if(flw.status === 'fulfilled' && flw.value){
      const d = flw.value;
      STATE_WEATHER.forecast = {
        desc: d.forecastDesc || null,
        icon: d.forecastIcon != null ? hkoIcon(d.forecastIcon) : null,
        outlook: d.outlook || null,
        generalSituation: d.generalSituation || null
      };
    }
    STATE_WEATHER.lastFetch = Date.now();
    STATE_WEATHER.loading = false;
    if(header) header.classList.remove('loading');
    // Re-render weather surfaces
    renderHeaderWeather();
    renderHomeWeather();
  }catch(e){
    STATE_WEATHER.error = e;
    STATE_WEATHER.loading = false;
    if(header) header.classList.remove('loading');
    // Hide weather on error (don't spam with toast)
    if(header) header.classList.add('error');
    const card = $('.weather-card');
    if(card) card.classList.add('error');
  }
}
```

- [ ] **Step 2: Add render functions for weather surfaces**

Insert right after the `fetchWeatherData()` function:

```js
function renderHeaderWeather(){
  const el = $('.weather-header');
  if(!el) return;
  if(!STATE_WEATHER.current || STATE_WEATHER.current.temp == null){
    el.classList.add('error');
    return;
  }
  el.classList.remove('error');
  const c = STATE_WEATHER.current;
  el.innerHTML = `
    <span class="wt-icon">${c.icon || '🌤️'}</span>
    <span class="wt-temp">${c.temp}°</span>
    ${c.humi != null ? `<span class="wt-humi"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M12 2.69 17.66 10A6 6 0 0 1 6.34 10Z"/></svg>${c.humi}%</span>` : ''}
  `;
}

function renderHomeWeather(){
  const card = $('.weather-card');
  if(!card) return;
  if(!STATE_WEATHER.current){
    card.classList.add('loading');
    return;
  }
  card.classList.remove('loading','error');
  const c = STATE_WEATHER.current;
  const f = STATE_WEATHER.forecast;
  // Build warnings HTML
  let warnHtml = '';
  if(c.rainstorm){
    const stormLabel = c.rainstorm.code === 'RAIN' ?
      (c.rainstorm.msg || '黃色暴雨警告信號') : (c.rainstorm.msg || c.rainstorm.code);
    const sev = 'amber';
    warnHtml += `<div class="wc-warn ${sev}">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0Z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
      ${escapeHTML(stormLabel)}
    </div>`;
  }
  if(c.tcInfo){
    warnHtml += `<div class="wc-warn red">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="10"/><path d="M12 2v20M2 12h20"/></svg>
      ${escapeHTML(c.tcInfo.tcname || c.tcInfo.msg || '熱帶氣旋')}
    </div>`;
  }
  // Build forecast text
  let forecastHtml = '';
  if(f?.desc || f?.outlook){
    forecastHtml = `<div class="wc-forecast">`;
    if(f.desc) forecastHtml += `📋 ${escapeHTML(f.desc)}`;
    if(f.outlook) forecastHtml += `<br>🔮 ${escapeHTML(f.outlook)}`;
    forecastHtml += `</div>`;
  }
  // UV label
  let uvLabel = '';
  if(c.uv != null){
    const lvls = ['低 Low','低 Low','低 Low','中等 Moderate','中等 Moderate','高 High','高 High','很高 Very High','很高 Very High','很高 Very High'];
    uvLabel = `${c.uv} ${lvls[Math.min(c.uv, 9)] || ''}`;
  }
  const updateTime = c.updateTime ? new Date(c.updateTime).toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit',hour12:false}) : '';
  card.innerHTML = `
    <div class="wc-head">
      <span class="wc-label">${c.icon || '🌤️'} 天氣 · 即時</span>
      <span class="wc-updated">${updateTime ? '更新 ' + updateTime : ''}</span>
    </div>
    <div class="wc-main">
      <div class="wc-temp">${c.temp != null ? c.temp : '--'}<span class="deg">°C</span></div>
      <div class="wc-details">
        <div class="wc-row"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M12 2.69 17.66 10A6 6 0 0 1 6.34 10Z"/></svg>濕度 Humidity <span class="val">${c.humi != null ? c.humi+'%' : '--'}</span></div>
        ${c.uv != null ? `<div class="wc-row"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><circle cx="12" cy="12" r="4"/><path d="M12 2v2M12 20v2M4.93 4.93l1.41 1.41M17.66 17.66l1.41 1.41M2 12h2M20 12h2M6.34 17.66l-1.41 1.41M17.66 6.34l1.41-1.41"/></svg>UV 指數 <span class="val">${uvLabel}</span></div>` : ''}
        ${c.rainfall != null ? `<div class="wc-row"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M12 2.69 17.66 10A6 6 0 0 1 6.34 10Z"/><path d="M8 14a3 3 0 0 0 3 3"/></svg>雨量 Rainfall <span class="val">${escapeHTML(c.rainMsg || c.rainfall+'mm')}</span></div>` : ''}
      </div>
    </div>
    ${warnHtml}
    ${forecastHtml}
  `;
}

function renderWeatherPanel(){
  const panel = $('#weatherPanel');
  if(!panel) return;
  const c = STATE_WEATHER.current;
  const f = STATE_WEATHER.forecast;
  if(!c){
    panel.innerHTML = `<div class="wp-handle"></div><div style="padding:30px;text-align:center;color:var(--ink-2)">載入中…<br>Loading</div>`;
    return;
  }
  let warnHtml = '';
  if(c.rainstorm){
    const sev = 'amber';
    warnHtml += `<div class="wp-warning ${sev}">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" style="width:18px;height:18px;flex-shrink:0"><path d="M10.29 3.86 1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0Z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>
      ${escapeHTML(c.rainstorm.msg || c.rainstorm.code || '天氣警告')}
    </div>`;
  }
  if(c.tcInfo){
    warnHtml += `<div class="wp-warning red">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" style="width:18px;height:18px;flex-shrink:0"><circle cx="12" cy="12" r="10"/><path d="M12 2v20M2 12h20"/></svg>
      ${escapeHTML(c.tcInfo.tcname || c.tcInfo.msg || '熱帶氣旋')}
    </div>`;
  }
  const updateTime = c.updateTime ? new Date(c.updateTime).toLocaleTimeString('en-GB',{hour:'2-digit',minute:'2-digit',hour12:false}) : '';
  panel.innerHTML = `
    <div class="wp-handle" onclick="closeWeatherPanel()"></div>
    <div class="wp-title">📍 香港天氣 Hong Kong Weather</div>
    <div class="wp-subtitle">${c.icon || ''} ${updateTime ? '更新 ' + updateTime : ''} · 資料來源: 香港天文台 HKO</div>
    ${warnHtml}
    <div class="wp-grid">
      <div class="wp-stat">
        <div class="lbl">溫度 Temperature</div>
        <div class="val">${c.temp != null ? c.temp : '--'}<span class="unit">°C</span></div>
      </div>
      <div class="wp-stat">
        <div class="lbl">濕度 Humidity</div>
        <div class="val">${c.humi != null ? c.humi : '--'}<span class="unit">%</span></div>
      </div>
      ${c.uv != null ? `<div class="wp-stat">
        <div class="lbl">UV 指數 UV Index</div>
        <div class="val">${c.uv} <span class="unit">${c.uvDesc || ''}</span></div>
      </div>` : ''}
      ${c.rainfall != null ? `<div class="wp-stat">
        <div class="lbl">雨量 Rainfall</div>
        <div class="val">${c.rainMsg ? escapeHTML(c.rainMsg) : c.rainfall}<span class="unit"></span></div>
      </div>` : ''}
    </div>
    ${(f?.desc || f?.outlook) ? `<div class="wp-forecast">
      <div style="font-family:var(--font-mono);font-size:9px;color:var(--neon-cyan);text-transform:uppercase;letter-spacing:0.14em;margin-bottom:6px">天氣預測 · Forecast</div>
      ${f.desc ? escapeHTML(f.desc) : ''}${f.outlook ? '<br><br>🔮 '+escapeHTML(f.outlook) : ''}
    </div>` : ''}
    <div style="margin-top:14px;text-align:center">
      <button class="retry" onclick="fetchWeatherData()" style="display:inline-flex;align-items:center;gap:6px">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" style="width:14px;height:14px"><path d="M21 12a9 9 0 1 1-3.5-7.1"/><path d="M21 4v5h-5"/></svg>
        重新整理 REFRESH
      </button>
    </div>
  `;
}

function openWeatherPanel(){
  const panel = $('#weatherPanel');
  if(!panel) return;
  renderWeatherPanel();
  panel.classList.add('show');
  try{ if(navigator.vibrate) navigator.vibrate(8); }catch(e){}
}

function closeWeatherPanel(){
  const panel = $('#weatherPanel');
  if(panel) panel.classList.remove('show');
}

function initWeather(){
  fetchWeatherData();
  STATE_WEATHER.interval = setInterval(fetchWeatherData, 600000); // 10 min
}
```

---

### Task 3: Add HTML Elements for Weather

**Files:**
- Modify: `src/index.html`

- [ ] **Step 1: Add weather header strip to the app header**

Find the header section (around line 1279). Insert the weather strip between the clock element and the theme toggle button.

Replace the line with the clock + weather + theme toggle:

**BEFORE:**
```html
    <div class="clock" id="clock"><span class="dot"></span><span id="clockTime">--:--</span></div>
    <button class="theme-toggle" id="themeToggle" aria-label="切換主題 Cycle theme" type="button">
```

**AFTER:**
```html
    <div class="clock" id="clock"><span class="dot"></span><span id="clockTime">--:--</span></div>
    <div class="weather-header error" id="weatherHeader" role="button" tabindex="0" aria-label="天氣資訊 Weather" title="點擊查看天氣詳情 Tap for weather details">
      <span class="wt-icon">🌤️</span>
      <span class="wt-temp">--°</span>
    </div>
    <button class="theme-toggle" id="themeToggle" aria-label="切換主題 Cycle theme" type="button">
```

- [ ] **Step 2: Add weather card to the home pane hero area**

Find the home pane section (around line 1314). Insert the weather card right after the `.modes` grid and before the `section-head` for MTR lines.

**Insert after the `</div>` closing the `.modes` section (after line 1392's `</button>` closing + `</div>` of .modes), before the `section-head` at line 1394:**

```html
      <!-- WEATHER CARD -->
      <div class="weather-card loading" id="weatherCard">
        <div class="wc-head">
          <span class="wc-label">🌤️ 天氣 · 即時</span>
        </div>
        <div class="wc-main">
          <div class="wc-temp">--<span class="deg">°C</span></div>
        </div>
      </div>
```

- [ ] **Step 3: Add weather slide-up panel near the bottom of the body**

Insert the weather panel HTML right before the lightbox div (before line 3730):

```html
<!-- WEATHER DETAIL PANEL -->
<div class="weather-panel" id="weatherPanel"></div>
```

---

### Task 4: Wire Weather Into App Boot

**Files:**
- Modify: `src/index.html`

- [ ] **Step 1: Add weather header click handler and init call to the init section**

In the INIT section (starting at line 3656), after the event handlers at the bottom (before the lightbox init at line 3723), add:

```js
/* WEATHER */
initWeather();
$('#weatherHeader').addEventListener('click', openWeatherPanel);
$('#weatherHeader').addEventListener('keydown', e=>{
  if(e.key==='Enter'||e.key===' '){e.preventDefault();openWeatherPanel()}
});
// Close weather panel on background overlay tap
$('#weatherPanel').addEventListener('click', e=>{
  if(e.target === e.currentTarget) closeWeatherPanel();
});
document.addEventListener('keydown', e=>{
  if(e.key==='Escape') closeWeatherPanel();
});
```

- [ ] **Step 2: Refresh weather when home pane is visited**

In the `goPane` function (around line 2075), find the existing line:

```js
if(pane==='home') renderHome();
```

Change it to:
```js
if(pane==='home'){ renderHome(); renderHomeWeather(); }
```

---

### Task 5: Verify & Test

- [ ] **Step 1: Check diagnostics**

Run: `npx lsp_diagnostics` on `src/index.html` — check for any JS syntax errors.

- [ ] **Step 2: Manual verification checklist**

1. Header weather strip shows temp + icon on page load (after a second for HKO API response)
2. Tapping weather strip opens the detail slide-up panel
3. Slide-up panel shows temp, humidity, UV, rainfall grid
4. Weather card is visible on the home page with temp and details
5. Weather card updates when data refreshes
6. Active weather warnings (rainstorm/tropical cyclone) display with amber/red styling
7. Forecast text appears when available
8. All weather elements gracefully hide on fetch error
9. Light/dark theme both look correct
10. 10-minute auto-refresh interval is running (check console for fetch calls)
