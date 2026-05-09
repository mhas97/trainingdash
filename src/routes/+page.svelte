<script>
  const ACTIVITY_COLORS = { run: '#ff6b35', gym: '#b44fff', cycle: '#00e5ff' };

  const EXAMPLE_ACTIVITIES = [
    { id: '1',  type: 'run',   date: '2026-02-18', distance: 3.2,  duration: '30:05', elevation:  42, heartrate: 145, notes: 'shaking off the rust' },
    { id: '2',  type: 'run',   date: '2026-02-20', distance: 4.8,  duration: '46:10', elevation:  58, heartrate: 150, notes: '' },
    { id: '37', type: 'gym',   date: '2026-02-21', distance: null, duration: '55:00', elevation: null, heartrate: 128, notes: 'upper body' },
    { id: '3',  type: 'run',   date: '2026-02-25', distance: 3.5,  duration: '33:20', elevation:  35, heartrate: 148, notes: 'windy' },
    { id: '38', type: 'cycle', date: '2026-02-26', distance: 10.2, duration: '38:10', elevation:  78, heartrate: 138, notes: 'canal path' },
    { id: '4',  type: 'run',   date: '2026-02-27', distance: 6.5,  duration: '1:02:40', elevation: 82, heartrate: 153, notes: 'first long run in a while' },
    { id: '39', type: 'gym',   date: '2026-03-02', distance: null, duration: '50:00', elevation: null, heartrate: 126, notes: '' },
    { id: '5',  type: 'run',   date: '2026-03-04', distance: 4.1,  duration: '39:15', elevation:  50, heartrate: 149, notes: '' },
    { id: '6',  type: 'run',   date: '2026-03-06', distance: 3.2,  duration: '30:50', elevation:  28, heartrate: 135, notes: 'recovery' },
    { id: '40', type: 'cycle', date: '2026-03-07', distance: 14.8, duration: '52:30', elevation: 118, heartrate: 142, notes: 'morning ride' },
    { id: '7',  type: 'run',   date: '2026-03-08', distance: 3.8,  duration: '36:20', elevation:  44, heartrate: 147, notes: '' },
    { id: '41', type: 'gym',   date: '2026-03-09', distance: null, duration: '45:00', elevation: null, heartrate: 136, notes: 'legs' },
    { id: '8',  type: 'run',   date: '2026-03-11', distance: 4.5,  duration: '43:00', elevation:  55, heartrate: 151, notes: '' },
    { id: '9',  type: 'run',   date: '2026-03-13', distance: 3.8,  duration: '36:45', elevation:  40, heartrate: 148, notes: 'rainy' },
    { id: '42', type: 'gym',   date: '2026-03-14', distance: null, duration: '1:00:00', elevation: null, heartrate: 132, notes: 'push day' },
    { id: '10', type: 'run',   date: '2026-03-15', distance: 4.7,  duration: '45:10', elevation:  64, heartrate: 155, notes: 'felt good' },
    { id: '43', type: 'cycle', date: '2026-03-16', distance: 18.5, duration: '1:05:20', elevation: 176, heartrate: 145, notes: '' },
    { id: '11', type: 'run',   date: '2026-03-18', distance: 4.2,  duration: '40:30', elevation:  52, heartrate: 150, notes: '' },
    { id: '12', type: 'run',   date: '2026-03-20', distance: 4.8,  duration: '46:00', elevation:  60, heartrate: 162, notes: 'tempo effort' },
    { id: '44', type: 'gym',   date: '2026-03-21', distance: null, duration: '52:00', elevation: null, heartrate: 130, notes: '' },
    { id: '13', type: 'run',   date: '2026-03-22', distance: 6.0,  duration: '58:30', elevation:  88, heartrate: 153, notes: 'long run' },
    { id: '14', type: 'run',   date: '2026-03-25', distance: 3.5,  duration: '33:45', elevation:  33, heartrate: 140, notes: 'cutback week' },
    { id: '45', type: 'gym',   date: '2026-03-27', distance: null, duration: '48:00', elevation: null, heartrate: 127, notes: 'pull day' },
    { id: '15', type: 'run',   date: '2026-03-28', distance: 4.2,  duration: '40:15', elevation:  48, heartrate: 149, notes: '' },
    { id: '16', type: 'run',   date: '2026-03-29', distance: 4.3,  duration: '41:20', elevation:  53, heartrate: 151, notes: '' },
    { id: '17', type: 'run',   date: '2026-04-01', distance: 4.5,  duration: '43:10', elevation:  58, heartrate: 152, notes: '' },
    { id: '46', type: 'gym',   date: '2026-04-02', distance: null, duration: '55:00', elevation: null, heartrate: 133, notes: '' },
    { id: '18', type: 'run',   date: '2026-04-03', distance: 5.2,  duration: '50:05', elevation:  68, heartrate: 158, notes: 'negative split' },
    { id: '47', type: 'cycle', date: '2026-04-04', distance: 22.1, duration: '1:18:00', elevation: 248, heartrate: 148, notes: 'long ride' },
    { id: '19', type: 'run',   date: '2026-04-05', distance: 7.3,  duration: '1:11:30', elevation: 104, heartrate: 156, notes: 'new longest run' },
    { id: '20', type: 'run',   date: '2026-04-08', distance: 4.8,  duration: '46:20', elevation:  63, heartrate: 152, notes: '' },
    { id: '48', type: 'gym',   date: '2026-04-08', distance: null, duration: '50:00', elevation: null, heartrate: 131, notes: 'double day' },
    { id: '21', type: 'run',   date: '2026-04-10', distance: 5.5,  duration: '53:00', elevation:  18, heartrate: 168, notes: 'track workout' },
    { id: '49', type: 'gym',   date: '2026-04-11', distance: null, duration: '45:00', elevation: null, heartrate: 129, notes: '' },
    { id: '22', type: 'run',   date: '2026-04-12', distance: 8.7,  duration: '1:24:45', elevation: 128, heartrate: 160, notes: 'crushed it' },
    { id: '50', type: 'cycle', date: '2026-04-13', distance: 16.3, duration: '58:40', elevation: 152, heartrate: 143, notes: '' },
    { id: '23', type: 'run',   date: '2026-04-15', distance: 5.0,  duration: '48:10', elevation:  68, heartrate: 153, notes: '' },
    { id: '51', type: 'gym',   date: '2026-04-16', distance: null, duration: '1:00:00', elevation: null, heartrate: 138, notes: 'legs' },
    { id: '24', type: 'run',   date: '2026-04-17', distance: 6.2,  duration: '1:00:20', elevation:  84, heartrate: 163, notes: 'first sub-hour 10k pace' },
    { id: '52', type: 'cycle', date: '2026-04-18', distance: 19.8, duration: '1:10:00', elevation: 318, heartrate: 150, notes: 'hilly route' },
    { id: '25', type: 'run',   date: '2026-04-19', distance: 4.8,  duration: '46:30', elevation:  60, heartrate: 151, notes: '' },
    { id: '26', type: 'run',   date: '2026-04-20', distance: 5.0,  duration: '48:00', elevation:  64, heartrate: 154, notes: 'back to back' },
    { id: '53', type: 'gym',   date: '2026-04-21', distance: null, duration: '55:00', elevation: null, heartrate: 127, notes: 'cutback week' },
    { id: '27', type: 'run',   date: '2026-04-22', distance: 4.0,  duration: '38:45', elevation:  38, heartrate: 142, notes: 'cutback week' },
    { id: '54', type: 'gym',   date: '2026-04-23', distance: null, duration: '48:00', elevation: null, heartrate: 130, notes: '' },
    { id: '28', type: 'run',   date: '2026-04-24', distance: 5.0,  duration: '48:20', elevation:  65, heartrate: 153, notes: '' },
    { id: '55', type: 'cycle', date: '2026-04-25', distance: 25.4, duration: '1:28:00', elevation: 276, heartrate: 152, notes: 'longest ride' },
    { id: '29', type: 'run',   date: '2026-04-27', distance: 7.0,  duration: '1:08:10', elevation:  98, heartrate: 155, notes: 'long run' },
    { id: '56', type: 'gym',   date: '2026-04-28', distance: null, duration: '52:00', elevation: null, heartrate: 132, notes: '' },
    { id: '30', type: 'run',   date: '2026-04-29', distance: 5.5,  duration: '53:15', elevation:  74, heartrate: 153, notes: '' },
    { id: '57', type: 'cycle', date: '2026-04-30', distance: 13.6, duration: '48:30', elevation:  88, heartrate: 135, notes: 'recovery spin' },
    { id: '31', type: 'run',   date: '2026-05-01', distance: 6.0,  duration: '58:00', elevation:  84, heartrate: 156, notes: 'May kicks off strong' },
    { id: '32', type: 'run',   date: '2026-05-02', distance: 4.3,  duration: '41:30', elevation:  54, heartrate: 150, notes: 'double day' },
    { id: '58', type: 'gym',   date: '2026-05-03', distance: null, duration: '1:00:00', elevation: null, heartrate: 134, notes: 'push day' },
    { id: '33', type: 'run',   date: '2026-05-04', distance: 7.2,  duration: '1:10:20', elevation: 108, heartrate: 162, notes: 'big effort' },
    { id: '34', type: 'run',   date: '2026-05-05', distance: 4.1,  duration: '39:30', elevation:  44, heartrate: 138, notes: 'recovery' },
    { id: '59', type: 'gym',   date: '2026-05-06', distance: null, duration: '50:00', elevation: null, heartrate: 129, notes: '' },
    { id: '35', type: 'run',   date: '2026-05-07', distance: 5.3,  duration: '51:10', elevation:  70, heartrate: 165, notes: 'tempo' },
    { id: '60', type: 'cycle', date: '2026-05-07', distance: 11.2, duration: '40:00', elevation:  92, heartrate: 140, notes: 'evening ride' },
    { id: '36', type: 'run',   date: '2026-05-08', distance: 4.5,  duration: '43:20', elevation:  58, heartrate: 152, notes: 'feeling fit' },
  ];

  const _stored = JSON.parse(localStorage.getItem('activities') ?? 'null');
  let _userActivities = $state(_stored ?? []);
  let showExample = $state(false);
  let activities = $derived(showExample ? EXAMPLE_ACTIVITIES : _userActivities);

  function toggleExample() {
    showExample = !showExample;
    if (showForm) showForm = false;
  }

  let showForm = $state(false);
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
      notes: form.notes
    });
    form = { type: form.type, date: new Date().toISOString().split('T')[0], distance: '', duration: '', elevation: '', heartrate: '', notes: '' };
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

  // ── Weekly chart — runs only ──────────────────────────────
  let chartWeeks = $derived.by(() => {
    const weeks = [];
    for (let i = 11; i >= 0; i--) {
      const ws = new Date(weekStart.getTime() - i * 7 * 86400000);
      const we = new Date(ws.getTime() + 7 * 86400000);
      const miles = activities
        .filter(a => {
          const d = new Date(a.date + 'T00:00:00');
          return d >= ws && d < we && (a.type ?? 'run') === 'run';
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

  let recentActivities = $derived.by(() => {
    const sorted = [...activities].sort((a, b) => {
      const byDate = b.date.localeCompare(a.date);
      return byDate !== 0 ? byDate : Number(b.id) - Number(a.id);
    });
    const filtered = filterType === 'all' ? sorted : sorted.filter(a => (a.type ?? 'run') === filterType);
    return filtered.slice(0, 15);
  });

  function fmtDate(ds) {
    return new Date(ds + 'T00:00:00').toLocaleDateString('en-GB', { day: 'numeric', month: 'short' });
  }
</script>

<div class="page">
  <div class="header">
    <div>
      <h1>training</h1>
      <p class="subtitle">activity log</p>
    </div>
    <div class="header-controls">
      <div class="unit-toggle">
        <button class:active={unit === 'mi'} onclick={() => unit = 'mi'}>mi</button>
        <button class:active={unit === 'km'} onclick={() => unit = 'km'}>km</button>
      </div>
      <button class="log-btn example-btn" class:active={showExample} onclick={toggleExample}>example</button>
      <button class="log-btn" onclick={() => (showForm = !showForm)}>
        {showForm ? 'cancel' : '+ log'}
      </button>
    </div>
  </div>

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
  <div class="section-label">this week</div>

  <!-- Weekly distance chart (runs) -->
  <div class="card">
    <div class="card-header">
      <span class="card-label">weekly running distance</span>
      {#if weekMiles > 0}<span class="card-value">{(weekMiles * kmFactor).toFixed(1)} {unit} this week</span>{/if}
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
          <line x1={PAD_X} y1={tick.y} x2={chartWidth - 8} y2={tick.y} stroke="#1e1e24" stroke-width="1" />
          <text x={PAD_X - 5} y={tick.y} fill="#555" font-size="9" text-anchor="end" dominant-baseline="middle">{tick.label}</text>
        {/each}
        <path d={areaPath} fill="rgba(255, 107, 53, 0.07)" />
        <polyline
          points={polylinePoints}
          fill="none"
          stroke="#ff6b35"
          stroke-width="1.5"
          stroke-opacity="0.5"
          stroke-linejoin="round"
        />
        {#if hoveredIdx !== null}
          <line
            x1={chartPoints[hoveredIdx].x} y1={PAD_Y}
            x2={chartPoints[hoveredIdx].x} y2={SVG_H - PAD_Y}
            stroke="#333" stroke-width="1"
          />
        {/if}
        {#each chartPoints as pt, i}
          {#if pt.miles > 0}
            <circle
              cx={pt.x} cy={pt.y}
              r={i === hoveredIdx ? 4 : pt.isCurrent ? 3.5 : 2.5}
              fill="#ff6b35"
              fill-opacity={i === hoveredIdx || pt.isCurrent ? 1 : 0.65}
            />
          {/if}
        {/each}
        {#each chartPoints as pt}
          <text x={pt.x} y={SVG_H + 12} fill="#555" font-size="9" text-anchor="middle">{pt.label}</text>
        {/each}
      </svg>
      {#if hoveredIdx !== null && chartPoints[hoveredIdx]?.miles > 0}
        <div class="chart-tooltip" style="left: {chartPoints[hoveredIdx].x}px; top: {chartPoints[hoveredIdx].y - 30}px">
          {(chartPoints[hoveredIdx].miles * kmFactor).toFixed(1)} {unit}
        </div>
      {/if}
    </div>
  </div>

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
      {#each calDays as day}
        {#if day === null}
          <div></div>
        {:else}
          {@const ds = toDateStr(calYear, calMonth, day)}
          {@const types = activityByDate[ds]}
          <div class="cal-day" class:is-today={ds === todayStr}
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
                    <span>{act.duration}</span>
                    <span class="cal-tip-notes">{act.notes || ''}</span>
                  {/each}
                </div>
              {/if}
            {/if}
          </div>
        {/if}
      {/each}
    </div>
  </div>

  <!-- Recent activity -->
  {#if recentActivities.length > 0 || filterType !== 'all'}
  <div class="card">
    <div class="list-header">
      <span class="card-label" style="margin-bottom: 0">recent activity</span>
      <div class="filter-tabs">
        <button class:active={filterType === 'all'} onclick={() => filterType = 'all'}>all</button>
        {#each ['run', 'gym', 'cycle'] as t}
          <button
            class:active={filterType === t}
            style={filterType === t ? `--tab-color: ${ACTIVITY_COLORS[t]}` : ''}
            onclick={() => filterType = t}
          >{t}</button>
        {/each}
      </div>
    </div>
    <div class="runs-list">
      <div class="run-header">
        <span class="run-date">date</span>
        <span class="run-type">type</span>
        <span class="run-dist">dist</span>
        <span class="run-dur">time</span>
        <span class="run-pace">vel</span>
        <span class="run-hr">hr</span>
        <span class="run-elev">elev</span>
        <span class="run-notes">notes</span>
      </div>
      {#each recentActivities as activity}
        {@const color = ACTIVITY_COLORS[activity.type ?? 'run']}
        <div class="run-row">
          <div class="run-date">{fmtDate(activity.date)}</div>
          <div class="run-type" style="color: {color}">{activity.type ?? 'run'}</div>
          <div class="run-dist">
            {activity.distance != null ? `${(activity.distance * kmFactor).toFixed(1)} ${unit}` : '—'}
          </div>
          {#if activity.duration}<div class="run-dur">{activity.duration}</div>{/if}
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
            onclick={() => {
              if (pendingDelete === activity.id) {
                deleteActivity(activity.id);
                pendingDelete = null;
              } else {
                pendingDelete = activity.id;
              }
            }}
            onblur={() => { if (pendingDelete === activity.id) pendingDelete = null; }}
          >{pendingDelete === activity.id ? 'del?' : '×'}</button>
        </div>
      {/each}
    </div>
  </div>
  {/if}
</div>

<style>
  .page {
    background: #0d0d0f;
    min-height: 100vh;
    padding: 2rem;
  }

  h1 {
    font-size: 40px;
    font-weight: 500;
    color: #fff;
    margin-bottom: 4px;
  }

  .subtitle {
    font-size: 15px;
    color: #888;
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
    align-items: center;
    gap: 8px;
    margin-top: 6px;
  }
  .unit-toggle {
    display: flex;
    border: 1px solid #333;
    border-radius: 6px;
    overflow: hidden;
  }
  .unit-toggle button {
    background: none;
    border: none;
    color: #555;
    font-size: 12px;
    padding: 6px 10px;
    cursor: pointer;
    letter-spacing: 0.04em;
  }
  .unit-toggle button.active {
    background: #222;
    color: #fff;
  }
  .log-btn {
    background: none;
    border: 1px solid #333;
    border-radius: 6px;
    color: #fff;
    font-size: 13px;
    padding: 7px 14px;
    cursor: pointer;
    letter-spacing: 0.04em;
  }
  .log-btn:hover { border-color: #555; }
  .example-btn.active { border-color: #ff6b35; color: #ff6b35; }
  /* ── Form ── */
  .form-card {
    background: #131318;
    border: 1px solid #222;
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
  label {
    display: flex;
    flex-direction: column;
    gap: 5px;
  }
  label span {
    font-size: 11px;
    color: #888;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  input {
    background: #0d0d0f;
    border: 1px solid #2a2a2a;
    border-radius: 6px;
    color: #fff;
    font-size: 14px;
    padding: 7px 10px;
    outline: none;
    font-family: inherit;
  }
  input:focus { border-color: #ff6b35; }
  .type-tabs {
    display: flex;
    border: 1px solid #2a2a2a;
    border-radius: 6px;
    overflow: hidden;
    height: 36px;
  }
  .type-tabs button {
    flex: 1;
    background: none;
    border: none;
    color: #555;
    font-size: 13px;
    cursor: pointer;
    font-family: inherit;
    letter-spacing: 0.04em;
  }
  .type-tabs button.active {
    background: #1e1e24;
    color: var(--tab-color, #ff6b35);
  }
  .submit-btn {
    background: var(--btn-color, #ff6b35);
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
  .submit-btn:disabled { background: #222; color: #555; cursor: default; }

  /* ── Stats ── */
  .stats-row {
    display: flex;
    background: #131318;
    border: 1px solid #222;
    border-radius: 12px;
    padding: 1.1rem 1.25rem;
    margin-bottom: 6px;
  }
  .stat {
    flex: 1;
    text-align: center;
  }
  .stat-value {
    font-size: 32px;
    font-weight: 500;
    color: #ff6b35;
    line-height: 1;
    margin-bottom: 4px;
  }
  .stat-label {
    font-size: 11px;
    color: #999;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  .divider {
    width: 1px;
    background: #444;
    margin: 0 4px;
  }
  .section-label {
    font-size: 11px;
    color: #777;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 1.25rem;
    padding-left: 2px;
  }

  /* ── Card ── */
  .card {
    background: #131318;
    border: 1px solid #222;
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
    color: #888;
    text-transform: uppercase;
    letter-spacing: 0.1em;
    margin-bottom: 14px;
  }
  .card-value {
    font-size: 12px;
    color: #ff6b35;
  }

  /* ── Calendar ── */
  .cal-nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 14px;
  }
  .cal-title {
    font-size: 15px;
    color: #fff;
    font-weight: 500;
  }
  .nav-btn {
    background: none;
    border: none;
    color: #555;
    font-size: 22px;
    cursor: pointer;
    padding: 0 6px;
    line-height: 1;
  }
  .nav-btn:hover { color: #fff; }
  .cal-grid {
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    gap: 3px;
  }
  .cal-label {
    text-align: center;
    font-size: 11px;
    color: #aaa;
    text-transform: uppercase;
    padding-bottom: 6px;
  }
  .cal-day {
    position: relative;
    aspect-ratio: 1;
    display: flex;
    flex-direction: column;
    align-items: center;
    border-top: 1px solid rgba(255, 107, 53, 0.15);
    justify-content: center;
    gap: 3px;
    font-size: 13px;
    color: #ccc;
    border-radius: 8px;
  }
  .cal-day.is-today {
    color: #fff;
    border: 1px solid #333;
  }
  .cal-dots {
    display: flex;
    gap: 2px;
  }
  .cal-tooltip {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-45%, -150%);
    background: #1a1a20;
    border: 1px solid #2a2a2a;
    border-radius: 8px;
    padding: 8px 10px;
    z-index: 20;
    pointer-events: none;
    white-space: nowrap;
    display: grid;
    grid-template-columns: 36px 52px 44px auto;
    column-gap: 8px;
    row-gap: 5px;
    align-items: center;
    font-size: 12px;
    color: #aaa;
  }
  .cal-tip-type {
    font-size: 10px;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    min-width: 36px;
  }
  .cal-tip-notes {
    color: #666;
  }
  .cal-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
  }

  /* ── Chart ── */
  .chart-container {
    position: relative;
    padding-bottom: 18px;
  }
  .chart-container svg {
    display: block;
    overflow: visible;
    cursor: crosshair;
  }
  .chart-tooltip {
    position: absolute;
    background: #1a1a20;
    border: 1px solid #333;
    border-radius: 4px;
    padding: 3px 8px;
    font-size: 11px;
    color: #ff6b35;
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
  .filter-tabs {
    display: flex;
    gap: 2px;
  }
  .filter-tabs button {
    background: none;
    border: 1px solid #2a2a2a;
    border-radius: 4px;
    color: #555;
    font-size: 11px;
    padding: 3px 8px;
    cursor: pointer;
    font-family: inherit;
    letter-spacing: 0.04em;
  }
  .filter-tabs button.active {
    border-color: var(--tab-color, #666);
    color: var(--tab-color, #fff);
  }
  .runs-list {
    display: flex;
    flex-direction: column;
  }
  .run-header {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 0 0 8px;
    border-bottom: 1px solid #1c1c1c;
    font-size: 10px;
    color: #444;
    text-transform: uppercase;
    letter-spacing: 0.08em;
  }
  .run-row {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 9px 0;
    border-bottom: 1px solid #1c1c1c;
    font-size: 13px;
  }
  .run-row:last-child { border-bottom: none; }
  .run-date { color: #aaa; min-width: 52px; }
  .run-type { color: #aaa; font-size: 11px; text-transform: uppercase; letter-spacing: 0.06em; min-width: 40px; }
  .run-dist { color: #aaa; min-width: 60px; }
  .run-pace { color: #aaa; font-size: 12px; min-width: 64px; }
  .run-hr { color: #aaa; font-size: 12px; min-width: 58px; }
  .run-dur { color: #aaa; min-width: 44px; }
  .run-elev { color: #aaa; font-size: 12px; min-width: 48px; }
  .run-notes {
    color: #aaa;
    flex: 1;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .del-btn {
    background: none;
    border: none;
    color: #2a2a2a;
    font-size: 18px;
    cursor: pointer;
    padding: 0 2px;
    line-height: 1;
    margin-left: auto;
  }
  .del-btn:hover { color: #ff4f7b; }
  .del-btn.del-armed { color: #ff4f7b; font-size: 12px; letter-spacing: 0.03em; }

  @media (max-width: 600px) {
    .page { padding: 1rem; }
    .form-row { grid-template-columns: 1fr; }
    .form-row label[style] { grid-column: 1 !important; }
    .runs-list { overflow-x: auto; }
    .run-header, .run-row { min-width: 580px; }
  }
</style>
