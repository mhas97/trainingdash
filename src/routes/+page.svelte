<script>
  import { EXAMPLE_ACTIVITIES } from '$lib/exampleData.js';

  const THEMES = {
    light:  { bg: '#f5f5f3', card: '#ffffff', card2: '#ebebea', hover: '#e2e2e0', rb: '#d8d8d6', b1: '#d8d8d6', b2: '#c8c8c6', b3: '#b0b0ae', track: '#a0a0a0', run: '#e05a1a', gym: '#8833cc', cycle: '#0088aa', tx0: '#111111', tx1: '#555555', tx2: '#999999' },
    dark:   { bg: '#0d0d0f', card: '#131318', card2: '#1a1a20', hover: '#1e1e24', rb: '#1c1c1c', b1: '#222',    b2: '#2a2a2a', b3: '#333',    track: '#444',    run: '#ff6b35', gym: '#b44fff', cycle: '#00e5ff', tx0: '#ffffff', tx1: '#aaaaaa', tx2: '#666666' },
    slate:  { bg: '#0b0c14', card: '#111420', card2: '#181b2e', hover: '#1e2138', rb: '#1a1d30', b1: '#1e2035', b2: '#252840', b3: '#2d3055', track: '#3a3d5c', run: '#6366f1', gym: '#f472b6', cycle: '#34d399', tx0: '#ffffff', tx1: '#aaaaaa', tx2: '#666666' },
    forest: { bg: '#080d09', card: '#0e160f', card2: '#131e14', hover: '#182419', rb: '#162017', b1: '#1a2e1b', b2: '#1f351f', b3: '#263d27', track: '#2d4a2e', run: '#4ade80', gym: '#facc15', cycle: '#38bdf8', tx0: '#ffffff', tx1: '#aaaaaa', tx2: '#666666' },
    amber:  { bg: '#0f0d0a', card: '#171410', card2: '#1e1a14', hover: '#23201a', rb: '#201e18', b1: '#2a2218', b2: '#322a1e', b3: '#3d3226', track: '#4a4030', run: '#f59e0b', gym: '#f87171', cycle: '#a78bfa', tx0: '#ffffff', tx1: '#aaaaaa', tx2: '#666666' },
    mono:   { bg: '#0a0a0a', card: '#111111', card2: '#181818', hover: '#1e1e1e', rb: '#1a1a1a', b1: '#222222', b2: '#2a2a2a', b3: '#333333', track: '#444444', run: '#e5e5e5', gym: '#888888', cycle: '#cccccc', tx0: '#ffffff', tx1: '#aaaaaa', tx2: '#666666' },
  };

  let theme = $state(localStorage.getItem('theme') ?? 'dark');
  let T = $derived(THEMES[theme] ?? THEMES.dark);
  let ACTIVITY_COLORS = $derived({ run: T.run, gym: T.gym, cycle: T.cycle });
  let themeStyle = $derived(
    `--bg:${T.bg};--card:${T.card};--card2:${T.card2};--hover:${T.hover};` +
    `--rb:${T.rb};--b1:${T.b1};--b2:${T.b2};--b3:${T.b3};--track:${T.track};` +
    `--c-run:${T.run};--c-gym:${T.gym};--c-cycle:${T.cycle};` +
    `--tx0:${T.tx0};--tx1:${T.tx1};--tx2:${T.tx2};`
  );
  $effect(() => { localStorage.setItem('theme', theme); });

  const _stored = JSON.parse(localStorage.getItem('activities') ?? 'null');
  let _userActivities = $state(_stored ?? []);
  let showExample = $state(false);
  let activities = $derived(showExample ? EXAMPLE_ACTIVITIES : _userActivities);

  function toggleExample() {
    showExample = !showExample;
    if (showForm) showForm = false;
  }

  let showForm = $state(false);
  let pendingTrack = $state(null);
  let mapActivityId = $state(null);
  let mapPos = $state(null);

  function toggleMap(e, activity) {
    if (!activity.track) return;
    if (mapActivityId === activity.id) { mapActivityId = null; mapPos = null; return; }
    const rect = e.currentTarget.getBoundingClientRect();
    const popH = 176;
    const top = rect.top > popH + 12 ? rect.top - popH - 8 : rect.bottom + 8;
    const left = Math.min(Math.max(8, rect.left), window.innerWidth - 296);
    mapActivityId = activity.id;
    mapPos = { top, left };
  }
  let form = $state({
    type: 'run',
    date: new Date().toISOString().split('T')[0],
    distance: '',
    duration: '',
    elevation: '',
    heartrate: '',
    notes: ''
  });

  let canSave = $derived(
    !!form.date &&
    !!form.duration &&
    (form.type === 'gym' || !!form.distance)
  );

  function normalizeDuration(val) {
    if (!val) return val;
    const parts = val.split(':');
    if (parts.length === 1) return parts[0].padStart(2, '0') + ':00';
    if (parts.length === 2) return parts[0].padStart(2, '0') + ':' + parts[1].padStart(2, '0');
    return parts[0] + ':' + parts[1].padStart(2, '0') + ':' + parts[2].padStart(2, '0');
  }

  function addActivity() {
    if (!canSave) return;
    const distanceMi = form.type === 'gym' ? null :
      unit === 'km' ? parseFloat(form.distance) / 1.60934 : parseFloat(form.distance);
    _userActivities.push({
      id: Date.now().toString(),
      type: form.type,
      date: form.date,
      distance: distanceMi,
      duration: normalizeDuration(form.duration),
      elevation: form.elevation ? parseFloat(form.elevation) : null,
      heartrate: form.heartrate ? parseInt(form.heartrate) : null,
      notes: form.notes,
      track: pendingTrack ?? undefined
    });
    form = { type: form.type, date: new Date().toISOString().split('T')[0], distance: '', duration: '', elevation: '', heartrate: '', notes: '' };
    pendingTrack = null;
    showForm = false;
  }

  function deleteActivity(id) {
    const idx = _userActivities.findIndex(a => a.id === id);
    if (idx !== -1) _userActivities.splice(idx, 1);
  }

  $effect(() => {
    if (!showExample) localStorage.setItem('activities', JSON.stringify(_userActivities));
  });

  // ── Calendar ──────────────────────────────────────────────
  const today = new Date();
  const todayStr = today.toISOString().split('T')[0];
  let calYear = $state(today.getFullYear());
  let calMonth = $state(today.getMonth());
  let hoveredDay = $state(null);
  let hoveredPb = $state(null);

  const MONTHS = ['January','February','March','April','May','June','July','August','September','October','November','December'];
  const DAYS = ['M','T','W','T','F','S','S'];

  function prevMonth() {
    if (calMonth === 0) { calMonth = 11; calYear--; } else calMonth--;
  }
  function nextMonth() {
    if (calMonth === 11) { calMonth = 0; calYear++; } else calMonth++;
  }

  let calDays = $derived.by(() => {
    const first = new Date(calYear, calMonth, 1);
    const last = new Date(calYear, calMonth + 1, 0);
    const cells = [];
    let dow = first.getDay();
    dow = dow === 0 ? 6 : dow - 1;
    for (let i = 0; i < dow; i++) cells.push(null);
    for (let d = 1; d <= last.getDate(); d++) cells.push(d);
    return cells;
  });

  let activityByDate = $derived(
    activities.reduce((acc, a) => {
      if (!acc[a.date]) acc[a.date] = new Set();
      acc[a.date].add(a.type ?? 'run');
      return acc;
    }, {})
  );

  function toDateStr(y, m, d) {
    return `${y}-${String(m + 1).padStart(2, '0')}-${String(d).padStart(2, '0')}`;
  }

  // ── This week ─────────────────────────────────────────────
  function getWeekStart(d) {
    const date = new Date(d);
    const dow = date.getDay();
    date.setDate(date.getDate() - (dow === 0 ? 6 : dow - 1));
    date.setHours(0, 0, 0, 0);
    return date;
  }

  const weekStart = getWeekStart(today);
  const weekEnd = new Date(weekStart.getTime() + 7 * 86400000);

  let thisWeekActivities = $derived(
    activities.filter(a => {
      const d = new Date(a.date + 'T00:00:00');
      return d >= weekStart && d < weekEnd;
    })
  );

  let weekMiles = $derived(
    +thisWeekActivities
      .filter(a => (a.type ?? 'run') === 'run')
      .reduce((s, a) => s + (a.distance || 0), 0)
      .toFixed(1)
  );

  function parseSecs(str) {
    if (!str) return 0;
    const p = str.split(':').map(Number);
    if (p.length === 3) return p[0] * 3600 + p[1] * 60 + p[2];
    if (p.length === 2) return p[0] * 60 + p[1];
    return 0;
  }

  let weekSecs = $derived(thisWeekActivities.reduce((s, a) => s + parseSecs(a.duration), 0));

  function calcPace(distanceMi, duration, kf) {
    if (!distanceMi || !duration) return null;
    const secs = parseSecs(duration);
    if (secs === 0) return null;
    const dist = distanceMi * kf;
    const spu = secs / dist;
    const m = Math.floor(spu / 60);
    const s = Math.round(spu % 60);
    return `${m}:${String(s).padStart(2, '0')}`;
  }

  function fmtTime(secs) {
    if (secs === 0) return '—';
    const h = Math.floor(secs / 3600);
    const m = Math.floor((secs % 3600) / 60);
    return h > 0 ? `${h}h ${m}m` : `${m}m`;
  }

  // ── Weekly chart ──────────────────────────────────────────
  let chartView = $state('run');
  let chartWeeks = $derived.by(() => {
    const weeks = [];
    for (let i = 11; i >= 0; i--) {
      const ws = new Date(weekStart.getTime() - i * 7 * 86400000);
      const we = new Date(ws.getTime() + 7 * 86400000);
      const miles = activities
        .filter(a => {
          const d = new Date(a.date + 'T00:00:00');
          return d >= ws && d < we && (a.type ?? 'run') === chartView;
        })
        .reduce((s, a) => s + (a.distance || 0), 0);
      const label = `${ws.getDate()}/${ws.getMonth() + 1}`;
      weeks.push({ miles: +miles.toFixed(1), label, isCurrent: i === 0 });
    }
    return weeks;
  });

  let maxMiles = $derived(Math.max(...chartWeeks.map(w => w.miles), 1));

  const SVG_H = 140, PAD_X = 28, PAD_Y = 10;
  let chartWidth = $state(300);
  let hoveredIdx = $state(null);
  let unit = $state('km');
  let kmFactor = $derived(unit === 'km' ? 1.60934 : 1);

  let chartPoints = $derived.by(() => {
    const usableW = chartWidth - PAD_X - 8;
    const usableH = SVG_H - PAD_Y * 2;
    return chartWeeks.map((week, i) => ({
      x: PAD_X + (i / (chartWeeks.length - 1)) * usableW,
      y: week.miles > 0 ? PAD_Y + (1 - week.miles / maxMiles) * usableH : PAD_Y + usableH,
      miles: week.miles,
      label: week.label,
      isCurrent: week.isCurrent
    }));
  });

  let polylinePoints = $derived(chartPoints.map(p => `${p.x.toFixed(1)},${p.y.toFixed(1)}`).join(' '));

  let areaPath = $derived.by(() => {
    if (!chartPoints.length) return '';
    const bottom = SVG_H - PAD_Y;
    return `M ${chartPoints.map(p => `${p.x.toFixed(1)},${p.y.toFixed(1)}`).join(' L ')} L ${chartPoints.at(-1).x.toFixed(1)},${bottom} L ${chartPoints[0].x.toFixed(1)},${bottom} Z`;
  });

  let yTicks = $derived([
    { label: `${(maxMiles * kmFactor).toFixed(0)}`, y: PAD_Y },
    { label: `${(maxMiles / 2 * kmFactor).toFixed(0)}`, y: PAD_Y + (SVG_H - PAD_Y * 2) / 2 },
  ]);

  function handleMouseMove(e) {
    const mouseX = e.clientX - e.currentTarget.getBoundingClientRect().left;
    let closest = 0, minDist = Infinity;
    chartPoints.forEach((pt, i) => {
      const d = Math.abs(pt.x - mouseX);
      if (d < minDist) { minDist = d; closest = i; }
    });
    hoveredIdx = closest;
  }

  // ── Recent activity ───────────────────────────────────────
  let filterType = $state('all');
  let pendingDelete = $state(null);
  let sortField = $state('date');
  let sortDir = $state('desc');

  function sortVal(a, field) {
    switch (field) {
      case 'date':  return a.date;
      case 'type':  return a.type ?? 'run';
      case 'dist':  return a.distance ?? null;
      case 'time':  return a.duration ? parseSecs(a.duration) : null;
      case 'pace':  return (a.distance && a.duration) ? parseSecs(a.duration) / a.distance : null;
      case 'hr':    return a.heartrate ?? null;
      case 'elev':  return a.elevation ?? null;
    }
  }

  let recentActivities = $derived.by(() => {
    const filtered = filterType === 'all' ? [...activities] : activities.filter(a => (a.type ?? 'run') === filterType);
    return filtered.sort((a, b) => {
      const av = sortVal(a, sortField), bv = sortVal(b, sortField);
      if (av === null && bv === null) return 0;
      if (av === null) return 1;
      if (bv === null) return -1;
      const cmp = typeof av === 'string' ? av.localeCompare(bv) : av - bv;
      return sortDir === 'desc' ? -cmp : cmp;
    });
  });

  function setSort(field) {
    if (sortField === field) sortDir = sortDir === 'desc' ? 'asc' : 'desc';
    else { sortField = field; sortDir = 'desc'; }
  }

  function sortIcon(field) {
    if (sortField !== field) return '↕';
    return sortDir === 'desc' ? '↓' : '↑';
  }

  function fmtDate(ds) {
    return new Date(ds + 'T00:00:00').toLocaleDateString('en-GB', { day: 'numeric', month: 'short' });
  }

  // ── Import (GPX / Garmin CSV) ─────────────────────────────
  let gpxInput;
  let csvInput;
  let importMsg = $state('');
  let showImportMenu = $state(false);
  let showThemeMenu = $state(false);

  function haversine(lat1, lon1, lat2, lon2) {
    const R = 6371000;
    const toRad = d => d * Math.PI / 180;
    const dLat = toRad(lat2 - lat1);
    const dLon = toRad(lon2 - lon1);
    const a = Math.sin(dLat/2)**2 + Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon/2)**2;
    return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  }

  function handleGpx(text) {
    const doc = new DOMParser().parseFromString(text, 'application/xml');
    const pts = [...doc.getElementsByTagName('trkpt')];
    if (pts.length < 2) return;

    const rawType = doc.getElementsByTagName('type')[0]?.textContent?.toLowerCase() ?? '';
    const type = rawType.includes('run') ? 'run'
      : (rawType.includes('cycl') || rawType.includes('bik')) ? 'cycle'
      : 'run';

    const firstTime = new Date(pts[0].getElementsByTagName('time')[0]?.textContent ?? '');
    const lastTime  = new Date(pts.at(-1).getElementsByTagName('time')[0]?.textContent ?? '');
    const date = firstTime.toISOString().split('T')[0];

    const totalSecs = Math.round((lastTime - firstTime) / 1000);
    const h = Math.floor(totalSecs / 3600);
    const m = Math.floor((totalSecs % 3600) / 60);
    const s = totalSecs % 60;
    const duration = h > 0
      ? `${h}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`
      : `${m}:${String(s).padStart(2,'0')}`;

    let distM = 0, elevGain = 0;
    let prevEle = parseFloat(pts[0].getElementsByTagName('ele')[0]?.textContent ?? '0');
    for (let i = 1; i < pts.length; i++) {
      const p = pts[i - 1], c = pts[i];
      distM += haversine(+p.getAttribute('lat'), +p.getAttribute('lon'), +c.getAttribute('lat'), +c.getAttribute('lon'));
      const ele = parseFloat(c.getElementsByTagName('ele')[0]?.textContent ?? '0');
      if (ele > prevEle) elevGain += ele - prevEle;
      prevEle = ele;
    }

    const hrEls = [...doc.getElementsByTagNameNS('*', 'hr')];
    const avgHr = hrEls.length
      ? Math.round(hrEls.reduce((s, el) => s + +el.textContent, 0) / hrEls.length)
      : null;

    const distKm = distM / 1000;
    const name = doc.getElementsByTagName('name')[0]?.textContent ?? '';

    const step = Math.max(1, Math.floor(pts.length / 400));
    const track = [];
    for (let i = 0; i < pts.length; i += step) {
      track.push([+pts[i].getAttribute('lat'), +pts[i].getAttribute('lon')]);
    }
    const lastPt = pts.at(-1);
    track.push([+lastPt.getAttribute('lat'), +lastPt.getAttribute('lon')]);
    pendingTrack = track;

    form.type      = type;
    form.date      = date;
    form.distance  = unit === 'km' ? distKm.toFixed(2) : (distKm / 1.60934).toFixed(2);
    form.duration  = duration;
    form.elevation = Math.round(unit === 'km' ? elevGain : elevGain * 3.28084).toString();
    form.heartrate = avgHr ? avgHr.toString() : '';
    form.notes     = name;
    showForm = true;
  }

  function parseCSVLine(line) {
    const result = [];
    let cur = '', inQuotes = false;
    for (const ch of line) {
      if (ch === '"') { inQuotes = !inQuotes; }
      else if (ch === ',' && !inQuotes) { result.push(cur); cur = ''; }
      else { cur += ch; }
    }
    result.push(cur);
    return result;
  }

  function fmtGarminDuration(str) {
    if (!str || str === '--') return null;
    const parts = str.split(':').map((p, i) => i === 2 ? Math.round(parseFloat(p)) : parseInt(p));
    if (parts.length === 3) {
      if (parts[0] === 0) return `${parts[1]}:${String(parts[2]).padStart(2,'0')}`;
      return `${parts[0]}:${String(parts[1]).padStart(2,'0')}:${String(parts[2]).padStart(2,'0')}`;
    }
    return str;
  }

  function handleCsv(text) {
    const TYPE_MAP = { 'running': 'run', 'strength training': 'gym', 'cycling': 'cycle', 'virtual ride': 'cycle' };
    const lines = text.split('\n').filter(l => l.trim());
    let added = 0;
    for (const line of lines.slice(1)) {
      const c = parseCSVLine(line);
      const type = TYPE_MAP[(c[0] ?? '').toLowerCase().trim()];
      if (!type) continue;
      const id = `garmin-${(c[1] ?? '').replace(' ', 'T')}`;
      if (_userActivities.some(a => a.id === id)) continue;
      const distKm = parseFloat(c[4]);
      const distance = (type === 'gym' || isNaN(distKm) || distKm === 0) ? null : distKm / 1.60934;
      const rawElev = (c[14] ?? '').replace(/,/g, '');
      const elevation = rawElev === '--' || rawElev === '' ? null : parseInt(rawElev);
      const avgHr = parseInt(c[7]);
      _userActivities.push({
        id,
        type,
        date: (c[1] ?? '').split(' ')[0],
        distance,
        duration: fmtGarminDuration(c[6]),
        elevation,
        heartrate: isNaN(avgHr) ? null : avgHr,
        notes: c[3] ?? '',
      });
      added++;
    }
    importMsg = `imported ${added} activities`;
    setTimeout(() => { importMsg = ''; }, 3000);
  }

  // ── Route map projection ──────────────────────────────────
  function routePolyline(track, W, H, pad) {
    const lats = track.map(p => p[0]);
    const lons = track.map(p => p[1]);
    const minLat = Math.min(...lats), maxLat = Math.max(...lats);
    const minLon = Math.min(...lons), maxLon = Math.max(...lons);
    const midLat = (minLat + maxLat) / 2;
    const cosLat = Math.cos(midLat * Math.PI / 180);
    const lonSpanCorr = (maxLon - minLon) * cosLat || 0.0001;
    const latSpan = maxLat - minLat || 0.0001;
    const innerW = W - pad * 2, innerH = H - pad * 2;
    const scale = Math.min(innerW / lonSpanCorr, innerH / latSpan);
    const offX = pad + (innerW - lonSpanCorr * scale) / 2;
    const offY = pad + (innerH - latSpan * scale) / 2;
    return track.map(([lat, lon]) =>
      `${(offX + (lon - minLon) * cosLat * scale).toFixed(1)},${(offY + (maxLat - lat) * scale).toFixed(1)}`
    ).join(' ');
  }

  function routeEndpoints(track, W, H, pad) {
    const all = routePolyline(track, W, H, pad).split(' ');
    const [sx, sy] = all[0].split(',').map(Number);
    const [ex, ey] = all.at(-1).split(',').map(Number);
    return { sx, sy, ex, ey };
  }

  // ── Annual totals ─────────────────────────────────────────
  const currentYear = today.getFullYear();
  let annualView = $state('run');

  let yearTypedActivities = $derived(
    activities.filter(a => a.date.startsWith(String(currentYear)) && (a.type ?? 'run') === annualView)
  );
  let yearDist = $derived(yearTypedActivities.reduce((s, a) => s + (a.distance || 0), 0));
  let yearElev = $derived(yearTypedActivities.reduce((s, a) => s + (a.elevation || 0), 0));
  let yearTotalSecs = $derived(yearTypedActivities.reduce((s, a) => s + parseSecs(a.duration), 0));
  let yearLongest = $derived(yearTypedActivities.reduce((b, a) => Math.max(b, a.distance || 0), 0));

  // ── All-time PBs ──────────────────────────────────────────
  const KM_TO_MI = 1 / 1.60934;
  const RUN_PB_TARGETS = [
    { label: '5k',       mi: 5 * KM_TO_MI },
    { label: '10k',      mi: 10 * KM_TO_MI },
    { label: 'half',     mi: 21.0975 * KM_TO_MI },
    { label: 'marathon', mi: 42.195 * KM_TO_MI },
  ];
  const CYCLE_PB_TARGETS = [
    { label: '5k',   mi: 5 * KM_TO_MI },
    { label: '10k',  mi: 10 * KM_TO_MI },
    { label: '50k',  mi: 50 * KM_TO_MI },
    { label: '100k', mi: 100 * KM_TO_MI },
  ];

  function bestEffortTime(type, targetMi) {
    let bestSpu = Infinity;
    let bestActivity = null;
    for (const a of activities) {
      if ((a.type ?? 'run') !== type || !a.distance || !a.duration) continue;
      if (a.distance < targetMi) continue;
      const spu = parseSecs(a.duration) / a.distance;
      if (spu < bestSpu) { bestSpu = spu; bestActivity = a; }
    }
    if (bestSpu === Infinity) return null;
    const totalSecs = Math.round(bestSpu * targetMi);
    const h = Math.floor(totalSecs / 3600);
    const m = Math.floor((totalSecs % 3600) / 60);
    const s = totalSecs % 60;
    const time = h > 0
      ? `${h}:${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`
      : `${m}:${String(s).padStart(2, '0')}`;
    const isEstimate = bestActivity.distance > targetMi * 1.005;
    return { time, isEstimate, source: bestActivity };
  }

  let pbTimes = $derived.by(() => {
    const targets = annualView === 'run' ? RUN_PB_TARGETS : CYCLE_PB_TARGETS;
    return targets.map(t => {
      const result = bestEffortTime(annualView, t.mi);
      if (!result) return { label: t.label, time: null, isEstimate: false, tooltip: null };
      let tooltip = null;
      if (result.isEstimate) {
        const src = result.source;
        const srcDist = src.distance * kmFactor;
        tooltip = `est. from ${srcDist.toFixed(1)} ${unit} on ${src.date} (${src.duration})`;
      }
      return { label: t.label, time: result.time, isEstimate: result.isEstimate, tooltip };
    });
  });
</script>

<div class="page" style={themeStyle}>
  <div class="header">
    <div>
      <h1>training</h1>
      <p class="subtitle">activity log</p>
    </div>
    <div class="header-controls">
      {#if showThemeMenu}
        <button class="import-overlay" onclick={() => showThemeMenu = false} aria-label="close theme menu"></button>
      {/if}
      <div class="import-wrap">
        <button class="log-btn" onclick={() => showThemeMenu = !showThemeMenu}>🎨</button>
        {#if showThemeMenu}
          <div class="import-menu">
            {#each Object.entries(THEMES) as [key, t]}
              <button
                class="theme-option"
                class:active={theme === key}
                onclick={() => { theme = key; showThemeMenu = false; }}
              >
                <span class="theme-dot" style="--swatch:{t.run}"></span>
                {key}
              </button>
            {/each}
          </div>
        {/if}
      </div>
      <button class="log-btn" onclick={() => unit = unit === 'km' ? 'mi' : 'km'}>{unit}</button>
      <button class="log-btn example-btn" class:active={showExample} onclick={toggleExample}>example</button>
      <input type="file" accept=".gpx" style="display:none" bind:this={gpxInput} onchange={e => { const f = e.target.files[0]; if (!f) return; const r = new FileReader(); r.onload = ev => handleGpx(ev.target.result); r.readAsText(f); e.target.value=''; }} />
      <input type="file" accept=".csv" style="display:none" bind:this={csvInput} onchange={e => { const f = e.target.files[0]; if (!f) return; const r = new FileReader(); r.onload = ev => handleCsv(ev.target.result); r.readAsText(f); e.target.value=''; }} />
      {#if showImportMenu}
        <button class="import-overlay" onclick={() => showImportMenu = false} aria-label="close menu"></button>
      {/if}
      <div class="import-wrap">
        <button class="log-btn" onclick={() => showImportMenu = !showImportMenu}>↑ import</button>
        {#if showImportMenu}
          <div class="import-menu">
            <button onclick={() => { gpxInput?.click(); showImportMenu = false; }}>gpx</button>
            <button onclick={() => { csvInput?.click(); showImportMenu = false; }}>activities</button>
          </div>
        {/if}
      </div>
      <button class="log-btn" onclick={() => (showForm = !showForm)}>
        {showForm ? 'cancel' : '+ log'}
      </button>
    </div>
  </div>

{#if importMsg}
  <div class="import-msg">{importMsg}</div>
{/if}

{#if showForm}
  <div class="form-card">
    <div class="form-row">
      <label>
        <span>type</span>
        <div class="type-tabs">
          {#each ['run', 'gym', 'cycle'] as t}
            <button
              class:active={form.type === t}
              style={form.type === t ? `--tab-color: ${ACTIVITY_COLORS[t]}` : ''}
              onclick={() => form.type = t}
            >{t}</button>
          {/each}
        </div>
      </label>
      <label>
        <span>date</span>
        <input type="date" bind:value={form.date} />
      </label>
    </div>
    <div class="form-row">
      {#if form.type !== 'gym'}
      <label>
        <span>distance ({unit})</span>
        <input type="number" step="0.01" min="0" placeholder="3.1" bind:value={form.distance} />
      </label>
      {/if}
      <label>
        <span>duration</span>
        <input type="text" placeholder="32:45" bind:value={form.duration} />
      </label>
    </div>
    <div class="form-row">
      {#if form.type !== 'gym'}
      <label>
        <span>elevation ({unit === 'km' ? 'm' : 'ft'}) — optional</span>
        <input type="number" step="1" min="0" placeholder="0" bind:value={form.elevation} />
      </label>
      {/if}
      <label style={form.type === 'gym' ? 'grid-column: 1 / -1' : ''}>
        <span>avg heart rate (bpm) — optional</span>
        <input type="number" step="1" min="0" placeholder="155" bind:value={form.heartrate} />
      </label>
    </div>
    <div class="form-row">
      <label style="grid-column: 1 / -1">
        <span>notes</span>
        <input type="text" placeholder="felt great..." bind:value={form.notes} />
      </label>
    </div>
    <button class="submit-btn" style="--btn-color: {ACTIVITY_COLORS[form.type]}" onclick={addActivity} disabled={!canSave}>
      save {form.type}
    </button>
  </div>
  {/if}

  <!-- This week stats -->
  <div class="stats-row">
    <div class="card-label" style="margin-bottom: 14px">this week</div>
    <div class="stats-items">
      <div class="stat">
        <div class="stat-value">{(weekMiles * kmFactor).toFixed(1)}</div>
        <div class="stat-label">{unit} running</div>
      </div>
      <div class="divider"></div>
      <div class="stat">
        <div class="stat-value">{fmtTime(weekSecs)}</div>
        <div class="stat-label">active time</div>
      </div>
      <div class="divider"></div>
      <div class="stat">
        <div class="stat-value">{thisWeekActivities.length}</div>
        <div class="stat-label">workouts</div>
      </div>
    </div>
  </div>

  <!-- Weekly distance chart -->
  <div class="card">
    <div class="card-header">
      <span class="card-label">weekly {chartView === 'run' ? 'running' : 'cycling'} distance</span>
      <div style="display:flex;align-items:center;gap:10px">
        {#if chartWeeks.at(-1)?.miles > 0}<span class="card-value">{(chartWeeks.at(-1).miles * kmFactor).toFixed(1)} {unit} this week</span>{/if}
        <div class="annual-toggle">
          <button class:active={chartView === 'run'} style={chartView === 'run' ? `--tab-color: ${ACTIVITY_COLORS.run}` : ''} onclick={() => chartView = 'run'}>run</button>
          <button class:active={chartView === 'cycle'} style={chartView === 'cycle' ? `--tab-color: ${ACTIVITY_COLORS.cycle}` : ''} onclick={() => chartView = 'cycle'}>cycle</button>
        </div>
      </div>
    </div>
    <div class="chart-container" bind:clientWidth={chartWidth}>
      <svg
        role="img"
        aria-label="Weekly mileage chart"
        width={chartWidth}
        height={SVG_H}
        onmousemove={handleMouseMove}
        onmouseleave={() => hoveredIdx = null}
      >
        {#each yTicks as tick}
          <line x1={PAD_X} y1={tick.y} x2={chartWidth - 8} y2={tick.y} style="stroke: var(--hover)" stroke-width="1" />
          <text x={PAD_X - 5} y={tick.y} style="fill: var(--tx2)" font-size="9" text-anchor="end" dominant-baseline="middle">{tick.label}</text>
        {/each}
        <path d={areaPath} style="fill: color-mix(in srgb, {ACTIVITY_COLORS[chartView]} 7%, transparent)" />
        <polyline
          points={polylinePoints}
          fill="none"
          style="stroke: {ACTIVITY_COLORS[chartView]}"
          stroke-width="1.5"
          stroke-opacity="0.5"
          stroke-linejoin="round"
        />
        {#if hoveredIdx !== null}
          <line
            x1={chartPoints[hoveredIdx].x} y1={PAD_Y}
            x2={chartPoints[hoveredIdx].x} y2={SVG_H - PAD_Y}
            style="stroke: var(--b3)" stroke-width="1"
          />
        {/if}
        {#each chartPoints as pt, i}
          {#if pt.miles > 0}
            <circle
              cx={pt.x} cy={pt.y}
              r={i === hoveredIdx ? 4 : pt.isCurrent ? 3.5 : 2.5}
              style="fill: var(--c-run)"
              fill-opacity={i === hoveredIdx || pt.isCurrent ? 1 : 0.65}
            />
          {/if}
        {/each}
        {#each chartPoints as pt}
          <text x={pt.x} y={SVG_H + 12} style="fill: var(--tx2)" font-size="9" text-anchor="middle">{pt.label}</text>
        {/each}
      </svg>
      {#if hoveredIdx !== null && chartPoints[hoveredIdx]?.miles > 0}
        <div class="chart-tooltip" style="left: {chartPoints[hoveredIdx].x}px; top: {chartPoints[hoveredIdx].y - 30}px">
          {(chartPoints[hoveredIdx].miles * kmFactor).toFixed(1)} {unit}
        </div>
      {/if}
    </div>
  </div>

  <!-- Annual totals + PBs -->
  <div class="card">
    <div class="annual-header">
      <span class="card-label" style="margin-bottom: 0">{currentYear} totals</span>
      <div class="annual-toggle">
        <button class:active={annualView === 'run'} style={annualView === 'run' ? `--tab-color: ${ACTIVITY_COLORS.run}` : ''} onclick={() => annualView = 'run'}>run</button>
        <button class:active={annualView === 'cycle'} style={annualView === 'cycle' ? `--tab-color: ${ACTIVITY_COLORS.cycle}` : ''} onclick={() => annualView = 'cycle'}>cycle</button>
      </div>
    </div>
    <div class="annual-grid" style="--av:{ACTIVITY_COLORS[annualView]}">
      <div class="annual-stat">
        <div class="annual-val">{(yearDist * kmFactor).toFixed(0)}</div>
        <div class="annual-lbl">{unit} {annualView === 'run' ? 'running' : 'cycling'}</div>
      </div>
      <div class="annual-stat">
        <div class="annual-val">{yearLongest > 0 ? (yearLongest * kmFactor).toFixed(1) : '—'}</div>
        <div class="annual-lbl">longest {annualView === 'run' ? 'run' : 'ride'} ({unit})</div>
      </div>
      <div class="annual-stat">
        <div class="annual-val">↑{yearElev.toFixed(0)}</div>
        <div class="annual-lbl">{unit === 'km' ? 'm' : 'ft'} elevation</div>
      </div>
      <div class="annual-stat">
        <div class="annual-val">{fmtTime(yearTotalSecs)}</div>
        <div class="annual-lbl">active time</div>
      </div>
      <div class="annual-stat">
        <div class="annual-val">{yearTypedActivities.length}</div>
        <div class="annual-lbl">workouts</div>
      </div>
    </div>
    <div class="pb-row">
      <span class="pb-label">all-time PBs</span>
      {#each pbTimes as pb}
        <span class="pb-item" class:pb-empty={!pb.time}>
          <span class="pb-key">{pb.label}</span>{pb.time ?? '—'}{#if pb.isEstimate}<button
            class="pb-est"
            onmouseenter={() => hoveredPb = pb.label}
            onmouseleave={() => hoveredPb = null}
            onclick={() => hoveredPb = hoveredPb === pb.label ? null : pb.label}
          >est.{#if hoveredPb === pb.label}<span class="pb-tooltip">{pb.tooltip}</span>{/if}</button>{/if}
        </span>
      {/each}
    </div>
  </div>

  <!-- Recent activity -->
  {#if recentActivities.length > 0 || filterType !== 'all'}
  <div class="card">
    <div class="list-header">
      <span class="card-label" style="margin-bottom: 0">activity</span>
      <div class="filter-tabs">
        <button class:active={filterType === 'all'} onclick={() => filterType = 'all'}>all</button>
        {#each ['run', 'cycle', 'gym'] as t}
          <button
            class:active={filterType === t}
            style={filterType === t ? `--tab-color: ${ACTIVITY_COLORS[t]}` : ''}
            onclick={() => filterType = t}
          >{t}</button>
        {/each}
      </div>
    </div>
    <div class="runs-list">
      <div class="runs-inner">
      <div class="run-header">
        <button class="sort-btn run-date"  class:sort-active={sortField==='date'}  onclick={() => setSort('date')}>date  <span class="sort-arrow" class:sort-arrow-dim={sortField!=='date'}>{sortIcon('date')}</span></button>
        <button class="sort-btn run-type"  class:sort-active={sortField==='type'}  onclick={() => setSort('type')}>type  <span class="sort-arrow" class:sort-arrow-dim={sortField!=='type'}>{sortIcon('type')}</span></button>
        <button class="sort-btn run-dist"  class:sort-active={sortField==='dist'}  onclick={() => setSort('dist')}>dist  <span class="sort-arrow" class:sort-arrow-dim={sortField!=='dist'}>{sortIcon('dist')}</span></button>
        <button class="sort-btn run-dur"   class:sort-active={sortField==='time'}  onclick={() => setSort('time')}>time  <span class="sort-arrow" class:sort-arrow-dim={sortField!=='time'}>{sortIcon('time')}</span></button>
        <button class="sort-btn run-pace"  class:sort-active={sortField==='pace'}  onclick={() => setSort('pace')}>pace  <span class="sort-arrow" class:sort-arrow-dim={sortField!=='pace'}>{sortIcon('pace')}</span></button>
        <button class="sort-btn run-hr"    class:sort-active={sortField==='hr'}    onclick={() => setSort('hr')}>hr    <span class="sort-arrow" class:sort-arrow-dim={sortField!=='hr'}>{sortIcon('hr')}</span></button>
        <button class="sort-btn run-elev"  class:sort-active={sortField==='elev'}  onclick={() => setSort('elev')}>elev  <span class="sort-arrow" class:sort-arrow-dim={sortField!=='elev'}>{sortIcon('elev')}</span></button>
        <span class="run-notes">notes</span>
      </div>
      <div class="runs-scroll">
        {#each recentActivities as activity}
          {@const color = ACTIVITY_COLORS[activity.type ?? 'run']}
          <div class="run-row" class:run-row-mapped={!!activity.track}
            role={activity.track ? 'button' : undefined}
            onclick={activity.track ? (e) => toggleMap(e, activity) : null}
            onkeydown={activity.track ? (e) => e.key === 'Enter' && toggleMap(e, activity) : null}>
            <div class="run-date">{fmtDate(activity.date)}</div>
            <div class="run-type" style="color: {color}">{activity.type ?? 'run'}</div>
            <div class="run-dist">
              {activity.distance != null ? `${(activity.distance * kmFactor).toFixed(1)} ${unit}` : '—'}
            </div>
            <div class="run-dur">{activity.duration || '—'}</div>
            <div class="run-pace">
              {#if (activity.type ?? 'run') !== 'gym'}
                {@const pace = calcPace(activity.distance, activity.duration, kmFactor)}
                {pace ? `${pace}/${unit}` : '—'}
              {:else}
                —
              {/if}
            </div>
            <div class="run-hr">
              {activity.heartrate ? `${activity.heartrate} bpm` : '—'}
            </div>
            <div class="run-elev">
              {activity.elevation != null ? `↑${activity.elevation}${unit === 'km' ? 'm' : 'ft'}` : '—'}
            </div>
            <div class="run-notes">{activity.notes || '—'}</div>
            <button
              class="del-btn"
              class:del-armed={pendingDelete === activity.id}
              onclick={(e) => { e.stopPropagation();
                if (pendingDelete === activity.id) {
                  deleteActivity(activity.id);
                  pendingDelete = null;
                } else {
                  pendingDelete = activity.id;
                }
              }}
              onblur={() => { if (pendingDelete === activity.id) pendingDelete = null; }}
            >{#if pendingDelete === activity.id}del?{:else}<svg width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"/><path d="M19 6l-1 14H6L5 6"/><path d="M10 11v6M14 11v6"/><path d="M9 6V4h6v2"/></svg>{/if}</button>
          </div>
        {/each}
      </div>
      </div>
    </div>
  </div>
  {/if}

  <!-- Calendar -->
  <div class="card">
    <div class="cal-nav">
      <button class="nav-btn" onclick={prevMonth}>‹</button>
      <span class="cal-title">{MONTHS[calMonth]} {calYear}</span>
      <button class="nav-btn" onclick={nextMonth}>›</button>
    </div>
    <div class="cal-grid">
      {#each DAYS as label}
        <div class="cal-label">{label}</div>
      {/each}
      {#each calDays as day, ci}
        {#if day === null}
          <div></div>
        {:else}
          {@const ds = toDateStr(calYear, calMonth, day)}
          {@const types = activityByDate[ds]}
          <div class="cal-day" class:is-today={ds === todayStr} class:tip-left={ci % 7 >= 4}
            role="gridcell"
            tabindex="0"
            onmouseenter={() => hoveredDay = ds}
            onmouseleave={() => hoveredDay = null}
          >
            <span>{day}</span>
            {#if types}
              <div class="cal-dots">
                {#each ['run','gym','cycle'] as t}
                  {#if types.has(t)}
                    <div class="cal-dot" style="background: {ACTIVITY_COLORS[t]}"></div>
                  {/if}
                {/each}
              </div>
            {/if}
            {#if hoveredDay === ds}
              {@const dayActivities = activities.filter(a => a.date === ds)}
              {#if dayActivities.length > 0}
                <div class="cal-tooltip">
                  {#each dayActivities as act}
                    <span class="cal-tip-type" style="color: {ACTIVITY_COLORS[act.type]}">{act.type}</span>
                    <span>{act.distance != null ? `${(act.distance * kmFactor).toFixed(1)}${unit}` : '—'}</span>
                    <span>{act.duration || '—'}</span>
                  {/each}
                </div>
              {/if}
            {/if}
          </div>
        {/if}
      {/each}
    </div>
  </div>
</div>

{#if mapActivityId && mapPos}
  {@const act = activities.find(a => a.id === mapActivityId)}
  {#if act?.track}
    {@const W = 280}
    {@const H = 160}
    {@const pad = 12}
    {@const pts = routePolyline(act.track, W, H, pad)}
    {@const ep = routeEndpoints(act.track, W, H, pad)}
    {@const col = ACTIVITY_COLORS[act.type ?? 'run']}
    <button class="map-backdrop" aria-label="Close map" onclick={() => { mapActivityId = null; mapPos = null; }}></button>
    <div class="map-popout" style="top:{mapPos.top}px;left:{mapPos.left}px">
      <svg viewBox="0 0 {W} {H}" width={W} height={H} style="display:block">
        <rect width={W} height={H} rx="10" fill="var(--card2)" />
        <polyline points={pts} fill="none" stroke={col} stroke-width="2" stroke-linejoin="round" stroke-linecap="round" opacity="0.85" />
        <circle cx={ep.sx} cy={ep.sy} r="5" fill={col} opacity="0.4" />
        <circle cx={ep.ex} cy={ep.ey} r="5" fill={col} />
      </svg>
    </div>
  {/if}
{/if}

<style>
  .page {
    background: var(--bg);
    min-height: 100vh;
    padding: 2rem;
    padding-bottom: max(2rem, calc(2rem + env(safe-area-inset-bottom)));
  }

  h1 {
    font-size: 40px;
    font-weight: 500;
    color: var(--tx0);
    margin-bottom: 4px;
  }

  .subtitle {
    font-size: 15px;
    color: var(--tx2);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: 1.75rem;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
  }

  .header-controls {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-end;
    align-items: center;
    gap: 8px;
    margin-top: 6px;
  }

  /* ── Theme menu ── */
  .theme-option {
    display: flex;
    align-items: center;
    gap: 8px;
    background: none;
    border: none;
    color: var(--tx1);
    font-size: 12px;
    padding: 7px 12px;
    cursor: pointer;
    text-align: left;
    border-radius: 4px;
    font-family: inherit;
    letter-spacing: 0.04em;
    width: 100%;
  }
  .theme-option:hover { background: var(--b1); color: var(--tx0); }
  .theme-option.active { color: var(--tx0); }
  .theme-dot {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: var(--swatch);
    flex-shrink: 0;
  }

  .log-btn {
    background: none;
    border: 1px solid var(--b3);
    border-radius: 6px;
    color: var(--tx0);
    font-size: 13px;
    padding: 7px 14px;
    cursor: pointer;
    letter-spacing: 0.04em;
  }
  .log-btn:hover { border-color: var(--tx2); }
  .example-btn.active { border-color: var(--c-run); color: var(--c-run); }
  .import-overlay {
    position: fixed;
    inset: 0;
    z-index: 99;
    background: none;
    border: none;
    cursor: default;
  }
  .import-wrap { position: relative; }
  .import-menu {
    position: absolute;
    right: 0;
    top: calc(100% + 4px);
    background: var(--card2);
    border: 1px solid var(--b3);
    border-radius: 8px;
    padding: 4px;
    z-index: 100;
    display: flex;
    flex-direction: column;
    min-width: 110px;
  }
  .import-menu button {
    background: none;
    border: none;
    color: var(--tx1);
    font-size: 12px;
    padding: 7px 12px;
    cursor: pointer;
    text-align: left;
    border-radius: 4px;
    font-family: inherit;
    letter-spacing: 0.04em;
  }
  .import-menu button:hover { background: var(--b1); color: var(--tx0); }
  .import-msg {
    background: var(--card);
    border: 1px solid var(--b2);
    border-radius: 8px;
    color: var(--tx1);
    font-size: 12px;
    padding: 8px 14px;
    margin-bottom: 12px;
    text-align: center;
  }

  /* ── Form ── */
  .form-card {
    background: var(--card);
    border: 1px solid var(--b1);
    border-radius: 12px;
    padding: 1.25rem;
    margin-bottom: 1.25rem;
  }
  .form-row {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 12px;
    margin-bottom: 12px;
  }
  label { display: flex; flex-direction: column; gap: 5px; }
  label span {
    font-size: 11px;
    color: var(--tx2);
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  input {
    background: var(--bg);
    border: 1px solid var(--b2);
    border-radius: 6px;
    color: var(--tx0);
    font-size: 14px;
    padding: 7px 10px;
    outline: none;
    font-family: inherit;
  }
  input:focus { border-color: var(--c-run); }
  .type-tabs {
    display: flex;
    border: 1px solid var(--b2);
    border-radius: 6px;
    overflow: hidden;
    height: 36px;
  }
  .type-tabs button {
    flex: 1;
    background: none;
    border: none;
    color: var(--tx2);
    font-size: 13px;
    cursor: pointer;
    font-family: inherit;
    letter-spacing: 0.04em;
  }
  .type-tabs button.active {
    background: var(--hover);
    color: var(--tab-color, var(--c-run));
  }
  .submit-btn {
    background: var(--btn-color, var(--c-run));
    border: none;
    border-radius: 6px;
    color: #fff;
    font-size: 13px;
    font-weight: 500;
    padding: 8px 20px;
    cursor: pointer;
    letter-spacing: 0.04em;
  }
  .submit-btn:hover:not(:disabled) { opacity: 0.85; }
  .submit-btn:disabled { background: var(--b1); color: var(--tx2); cursor: default; }

  /* ── Stats ── */
  .stats-row {
    display: flex;
    flex-direction: column;
    background: var(--card);
    border: 1px solid var(--b1);
    border-radius: 12px;
    padding: 1.1rem 1.25rem;
    margin-bottom: 12px;
  }
  .stats-items { display: flex; }
  .stat { flex: 1; text-align: center; }
  .stat-value {
    font-size: 32px;
    font-weight: 500;
    color: var(--c-run);
    line-height: 1;
    margin-bottom: 4px;
  }
  .stat-label {
    font-size: 11px;
    color: var(--tx2);
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  .divider { width: 1px; background: var(--track); margin: 0 4px; }

  /* ── Card ── */
  .card {
    background: var(--card);
    border: 1px solid var(--b1);
    border-radius: 12px;
    padding: 1.1rem 1.25rem;
    margin-bottom: 12px;
  }
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    margin-bottom: 14px;
  }
  .card-label {
    font-size: 11px;
    color: var(--tx2);
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 14px;
  }
  .card-value { font-size: 12px; color: var(--c-run); }

  /* ── Calendar ── */
  .cal-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 14px;
  }
  .cal-title { font-size: 15px; color: var(--tx0); font-weight: 500; }
  .nav-btn {
    background: none;
    border: none;
    color: var(--tx2);
    font-size: 22px;
    cursor: pointer;
    padding: 0 6px;
    line-height: 1;
  }
  .nav-btn:hover { color: var(--tx0); }
  .cal-grid { display: grid; grid-template-columns: repeat(7, 1fr); gap: 3px; }
  .cal-label {
    text-align: center;
    font-size: 11px;
    color: var(--tx2);
    text-transform: uppercase;
    padding-bottom: 6px;
  }
  .cal-day {
    position: relative;
    aspect-ratio: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    border-top: 1px solid color-mix(in srgb, var(--c-run) 15%, transparent);
    justify-content: center;
    gap: 3px;
    font-size: 13px;
    color: var(--tx1);
    border-radius: 8px;
  }
  .cal-day.is-today { color: var(--tx0); border: 1px solid var(--b3); }
  @media (hover: hover) {
    .cal-day { transition: background 0.15s ease; }
    .cal-day:hover { background: var(--hover); color: var(--tx0); }
  }
  .cal-dots { display: flex; gap: 2px; }
  .cal-tooltip {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -150%);
    background: var(--card2);
    border: 1px solid var(--b2);
    border-radius: 8px;
    padding: 8px 10px;
    z-index: 20;
    pointer-events: none;
    white-space: nowrap;
    display: grid;
    grid-template-columns: 36px 52px 44px;
    column-gap: 8px;
    row-gap: 5px;
    align-items: center;
    font-size: 12px;
    color: var(--tx1);
  }
  .cal-tip-type { font-size: 10px; text-transform: uppercase; letter-spacing: 0.06em; min-width: 36px; }
  .cal-dot { width: 6px; height: 6px; border-radius: 50%; }

  /* ── Chart ── */
  .chart-container { position: relative; padding-bottom: 18px; }
  .chart-container svg { display: block; overflow: visible; cursor: crosshair; }
  .chart-tooltip {
    position: absolute;
    background: var(--card2);
    border: 1px solid var(--b3);
    border-radius: 4px;
    padding: 3px 8px;
    font-size: 11px;
    color: var(--c-run);
    pointer-events: none;
    transform: translateX(-50%);
    white-space: nowrap;
  }

  /* ── Activity list ── */
  .list-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 14px;
  }
  .filter-tabs { display: flex; gap: 2px; }
  .filter-tabs button {
    background: none;
    border: 1px solid var(--b2);
    border-radius: 4px;
    color: var(--tx2);
    font-size: 11px;
    padding: 3px 8px;
    cursor: pointer;
    font-family: inherit;
    letter-spacing: 0.04em;
  }
  .filter-tabs button.active {
    border-color: var(--tab-color, var(--tx2));
    color: var(--tab-color, var(--tx0));
  }
  .runs-list { display: flex; flex-direction: column; }
  .runs-inner { display: flex; flex-direction: column; }
  .runs-scroll {
    max-height: 280px;
    overflow-y: auto;
    scrollbar-width: thin;
    scrollbar-color: var(--b2) transparent;
  }
  .run-header {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 0 0 8px;
    border-bottom: 1px solid var(--rb);
    font-size: 10px;
    color: var(--track);
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  .sort-btn {
    background: none; border: none; padding: 0; margin: 0;
    font: inherit; font-size: 10px; letter-spacing: 0.08em; text-transform: uppercase;
    color: var(--track); cursor: pointer; display: flex; align-items: center; gap: 2px;
  }
  .sort-btn:hover { color: var(--tx1); }
  .sort-btn.sort-active { color: var(--tx0); }
  .sort-arrow { font-size: 9px; opacity: 0.8; }
  .sort-arrow-dim { opacity: 0.25; }
  .run-row-mapped { cursor: pointer; }
  .map-backdrop { position: fixed; inset: 0; z-index: 40; background: transparent; border: none; padding: 0; cursor: default; }
  .map-popout {
    position: fixed; z-index: 50;
    border-radius: 10px; overflow: hidden;
    box-shadow: 0 4px 32px color-mix(in srgb, var(--bg) 40%, #000);
    border: 1px solid var(--b2);
    pointer-events: none;
  }
  .run-row {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 9px 0;
    border-bottom: 1px solid var(--rb);
    font-size: 13px;
  }
  .run-row:last-child { border-bottom: none; }
  @media (hover: hover) {
    .run-row { transition: background 0.15s ease; border-radius: 6px; }
    .run-row:hover { background: var(--hover); }
  }
  .run-date, .run-type, .run-dist, .run-pace, .run-hr, .run-dur, .run-elev {
    color: var(--tx1);
    flex: 0 0 68px;
    width: 68px;
    overflow: hidden;
    white-space: nowrap;
  }
  .run-type { font-size: 11px; text-transform: uppercase; letter-spacing: 0.06em; }
  .run-notes { color: var(--tx1); flex: 1; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .del-btn {
    background: none;
    border: none;
    color: #ff4f7b;
    font-size: 12px;
    cursor: pointer;
    padding: 0 2px;
    line-height: 1;
    margin-left: auto;
    opacity: 0.25;
  }
  .del-btn:hover { opacity: 1; }
  .del-btn.del-armed { opacity: 1; letter-spacing: 0.03em; }

  /* ── Annual + PBs ── */
  .annual-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 14px;
  }
  .annual-toggle {
    display: flex;
    border: 1px solid var(--b2);
    border-radius: 6px;
    overflow: hidden;
  }
  .annual-toggle button {
    background: none;
    border: none;
    color: var(--tx2);
    font-size: 11px;
    padding: 4px 10px;
    cursor: pointer;
    font-family: inherit;
    letter-spacing: 0.04em;
  }
  .annual-toggle button.active { background: var(--hover); color: var(--tab-color, var(--tx0)); }
  .annual-grid { display: flex; margin-bottom: 14px; }
  .annual-stat {
    flex: 1;
    text-align: center;
    border-right: 1px solid var(--b1);
    padding: 0 6px;
  }
  .annual-stat:first-child { padding-left: 0; }
  .annual-stat:last-child { border-right: none; padding-right: 0; }
  .annual-val { font-size: 22px; font-weight: 500; color: var(--av, var(--c-run)); line-height: 1; margin-bottom: 4px; }
  .annual-lbl { font-size: 10px; color: var(--tx2); text-transform: uppercase; letter-spacing: 0.08em; }
  .pb-row {
    display: flex;
    align-items: center;
    gap: 16px;
    border-top: 1px solid var(--rb);
    padding-top: 12px;
    flex-wrap: wrap;
  }
  .pb-label { font-size: 10px; color: var(--tx2); text-transform: uppercase; letter-spacing: 0.1em; }
  .pb-item { font-size: 12px; color: var(--tx1); position: relative; }
  .pb-key { font-size: 10px; color: var(--tx2); text-transform: uppercase; letter-spacing: 0.06em; margin-right: 5px; }
  .pb-empty { color: var(--track); }
  .pb-est {
    background: none;
    border: none;
    border-bottom: 1px dotted var(--tx2);
    padding: 0;
    font-family: inherit;
    font-size: 9px;
    color: var(--tx2);
    text-transform: uppercase;
    letter-spacing: 0.06em;
    margin-left: 3px;
    cursor: help;
    position: relative;
  }
  .pb-tooltip {
    position: absolute;
    bottom: calc(100% + 6px);
    left: 50%;
    transform: translateX(-50%);
    background: var(--card2);
    border: 1px solid var(--b2);
    border-radius: 8px;
    padding: 6px 10px;
    font-size: 11px;
    color: var(--tx1);
    width: max-content;
    max-width: min(300px, calc(100vw - 2rem));
    white-space: normal;
    z-index: 20;
    pointer-events: none;
    letter-spacing: 0;
    text-transform: none;
  }

  @media (max-width: 600px) {
    .page { padding: 1rem; padding-bottom: max(1rem, calc(1rem + env(safe-area-inset-bottom))); }
    .header { flex-direction: column; }
    .header-controls { width: 100%; justify-content: center; margin-bottom: 1rem; }
    .form-row { grid-template-columns: 1fr; }
    .form-row label[style] { grid-column: 1 !important; }
    .import-menu { left: 0; right: auto; }
    .tip-left .cal-tooltip { left: auto; right: 0; transform: translateY(-150%); }
    .runs-list { overflow-x: auto; scrollbar-width: none; }
    .runs-list::-webkit-scrollbar { display: none; }
    .runs-inner { min-width: 640px; }
    .runs-scroll { scrollbar-width: none; }
    .run-header, .run-row { min-width: unset; }
    .annual-grid { overflow-x: auto; scrollbar-width: none; }
    .annual-grid::-webkit-scrollbar { display: none; }
    .annual-stat { min-width: 90px; flex-shrink: 0; }
  }
</style>
