<script>
  const TEST_RUNS = [
    { id: '1',  date: '2026-02-18', distance: 3.2, duration: '30:05', notes: 'shaking off the rust' },
    { id: '2',  date: '2026-02-20', distance: 4.8, duration: '46:10', notes: '' },
    { id: '3',  date: '2026-02-25', distance: 3.5, duration: '33:20', notes: 'windy' },
    { id: '4',  date: '2026-02-27', distance: 6.5, duration: '1:02:40', notes: 'first long run in a while' },
    { id: '5',  date: '2026-03-04', distance: 4.1, duration: '39:15', notes: '' },
    { id: '6',  date: '2026-03-06', distance: 3.2, duration: '30:50', notes: 'recovery' },
    { id: '7',  date: '2026-03-08', distance: 3.8, duration: '36:20', notes: '' },
    { id: '8',  date: '2026-03-11', distance: 4.5, duration: '43:00', notes: '' },
    { id: '9',  date: '2026-03-13', distance: 3.8, duration: '36:45', notes: 'rainy' },
    { id: '10', date: '2026-03-15', distance: 4.7, duration: '45:10', notes: 'felt good' },
    { id: '11', date: '2026-03-18', distance: 4.2, duration: '40:30', notes: '' },
    { id: '12', date: '2026-03-20', distance: 4.8, duration: '46:00', notes: 'tempo effort' },
    { id: '13', date: '2026-03-22', distance: 6.0, duration: '58:30', notes: 'long run' },
    { id: '14', date: '2026-03-25', distance: 3.5, duration: '33:45', notes: 'cutback week' },
    { id: '15', date: '2026-03-28', distance: 4.2, duration: '40:15', notes: '' },
    { id: '16', date: '2026-03-29', distance: 4.3, duration: '41:20', notes: '' },
    { id: '17', date: '2026-04-01', distance: 4.5, duration: '43:10', notes: '' },
    { id: '18', date: '2026-04-03', distance: 5.2, duration: '50:05', notes: 'negative split' },
    { id: '19', date: '2026-04-05', distance: 7.3, duration: '1:11:30', notes: 'new longest run' },
    { id: '20', date: '2026-04-08', distance: 4.8, duration: '46:20', notes: '' },
    { id: '21', date: '2026-04-10', distance: 5.5, duration: '53:00', notes: 'track workout' },
    { id: '22', date: '2026-04-12', distance: 8.7, duration: '1:24:45', notes: 'crushed it' },
    { id: '23', date: '2026-04-15', distance: 5.0, duration: '48:10', notes: '' },
    { id: '24', date: '2026-04-17', distance: 6.2, duration: '1:00:20', notes: 'first sub-hour 10k pace' },
    { id: '25', date: '2026-04-19', distance: 4.8, duration: '46:30', notes: '' },
    { id: '26', date: '2026-04-20', distance: 5.0, duration: '48:00', notes: 'back to back' },
    { id: '27', date: '2026-04-22', distance: 4.0, duration: '38:45', notes: 'cutback week' },
    { id: '28', date: '2026-04-24', distance: 5.0, duration: '48:20', notes: '' },
    { id: '29', date: '2026-04-27', distance: 7.0, duration: '1:08:10', notes: 'long run' },
    { id: '30', date: '2026-04-29', distance: 5.5, duration: '53:15', notes: '' },
    { id: '31', date: '2026-05-01', distance: 6.0, duration: '58:00', notes: 'May kicks off strong' },
    { id: '32', date: '2026-05-02', distance: 4.3, duration: '41:30', notes: 'double day' },
    { id: '33', date: '2026-05-04', distance: 7.2, duration: '1:10:20', notes: 'big effort' },
    { id: '34', date: '2026-05-05', distance: 4.1, duration: '39:30', notes: 'recovery' },
    { id: '35', date: '2026-05-07', distance: 5.3, duration: '51:10', notes: 'tempo' },
    { id: '36', date: '2026-05-08', distance: 4.5, duration: '43:20', notes: 'feeling fit' },
  ];

  const _stored = JSON.parse(localStorage.getItem('runs') ?? 'null');
  let runs = $state((_stored?.length > 0) ? _stored : TEST_RUNS);

  let showForm = $state(false);
  let form = $state({
    date: new Date().toISOString().split('T')[0],
    distance: '',
    duration: '',
    notes: ''
  });

  function addRun() {
    if (!form.date || !form.distance) return;
    const distanceMi = unit === 'km'
      ? parseFloat(form.distance) / 1.60934
      : parseFloat(form.distance);
    runs.push({
      id: Date.now().toString(),
      date: form.date,
      distance: distanceMi,
      duration: form.duration,
      notes: form.notes
    });
    form = { date: new Date().toISOString().split('T')[0], distance: '', duration: '', notes: '' };
    showForm = false;
  }

  function deleteRun(id) {
    const idx = runs.findIndex(r => r.id === id);
    if (idx !== -1) runs.splice(idx, 1);
  }

  $effect(() => {
    localStorage.setItem('runs', JSON.stringify(runs));
  });

  // ── Calendar ──────────────────────────────────────────────
  const today = new Date();
  const todayStr = today.toISOString().split('T')[0];
  let calYear = $state(today.getFullYear());
  let calMonth = $state(today.getMonth());

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

  let runDates = $derived(new Set(runs.map(r => r.date)));

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

  let thisWeekRuns = $derived(
    runs.filter(r => {
      const d = new Date(r.date + 'T00:00:00');
      return d >= weekStart && d < weekEnd;
    })
  );

  let weekMiles = $derived(+thisWeekRuns.reduce((s, r) => s + r.distance, 0).toFixed(1));

  function parseSecs(str) {
    if (!str) return 0;
    const p = str.split(':').map(Number);
    if (p.length === 3) return p[0] * 3600 + p[1] * 60 + p[2];
    if (p.length === 2) return p[0] * 60 + p[1];
    return 0;
  }

  let weekSecs = $derived(thisWeekRuns.reduce((s, r) => s + parseSecs(r.duration), 0));

  function fmtTime(secs) {
    if (secs === 0) return '—';
    const h = Math.floor(secs / 3600);
    const m = Math.floor((secs % 3600) / 60);
    return h > 0 ? `${h}h ${m}m` : `${m}m`;
  }

  // ── Weekly chart (last 12 weeks) ──────────────────────────
  let chartWeeks = $derived.by(() => {
    const weeks = [];
    for (let i = 11; i >= 0; i--) {
      const ws = new Date(weekStart.getTime() - i * 7 * 86400000);
      const we = new Date(ws.getTime() + 7 * 86400000);
      const miles = runs
        .filter(r => { const d = new Date(r.date + 'T00:00:00'); return d >= ws && d < we; })
        .reduce((s, r) => s + r.distance, 0);
      const label = `${ws.getDate()}/${ws.getMonth() + 1}`;
      weeks.push({ miles: +miles.toFixed(1), label, isCurrent: i === 0 });
    }
    return weeks;
  });

  let maxMiles = $derived(Math.max(...chartWeeks.map(w => w.miles), 1));

  const SVG_H = 140, PAD_X = 28, PAD_Y = 10;
  let chartWidth = $state(300);
  let hoveredIdx = $state(null);
  let unit = $state('mi');
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

  // ── Recent runs ───────────────────────────────────────────
  let recentRuns = $derived([...runs].sort((a, b) => {
    const byDate = b.date.localeCompare(a.date);
    return byDate !== 0 ? byDate : Number(b.id) - Number(a.id);
  }).slice(0, 15));

  function fmtDate(ds) {
    return new Date(ds + 'T00:00:00').toLocaleDateString('en-GB', { day: 'numeric', month: 'short' });
  }
</script>

<div class="page">
  <div class="header">
    <div>
      <h1>runs</h1>
      <p class="subtitle">training log</p>
    </div>
    <div class="header-controls">
      <div class="unit-toggle">
        <button class:active={unit === 'mi'} onclick={() => unit = 'mi'}>mi</button>
        <button class:active={unit === 'km'} onclick={() => unit = 'km'}>km</button>
      </div>
      <button class="log-btn" onclick={() => (showForm = !showForm)}>
        {showForm ? 'cancel' : '+ log run'}
      </button>
    </div>
  </div>

  {#if showForm}
  <div class="form-card">
    <div class="form-row">
      <label>
        <span>date</span>
        <input type="date" bind:value={form.date} />
      </label>
      <label>
        <span>distance ({unit})</span>
        <input type="number" step="0.01" min="0" placeholder="3.1" bind:value={form.distance} />
      </label>
    </div>
    <div class="form-row">
      <label>
        <span>duration</span>
        <input type="text" placeholder="32:45" bind:value={form.duration} />
      </label>
      <label>
        <span>notes</span>
        <input type="text" placeholder="felt great..." bind:value={form.notes} />
      </label>
    </div>
    <button class="submit-btn" onclick={addRun}>save run</button>
  </div>
  {/if}

  <!-- This week stats -->
  <div class="stats-row">
    <div class="stat">
      <div class="stat-value">{(weekMiles * kmFactor).toFixed(1)}</div>
      <div class="stat-label">{unit}</div>
    </div>
    <div class="divider"></div>
    <div class="stat">
      <div class="stat-value">{fmtTime(weekSecs)}</div>
      <div class="stat-label">time</div>
    </div>
    <div class="divider"></div>
    <div class="stat">
      <div class="stat-value">{thisWeekRuns.length}</div>
      <div class="stat-label">runs</div>
    </div>
  </div>
  <div class="section-label">this week</div>

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
          <div
            class="cal-day"
            class:has-run={runDates.has(ds)}
            class:is-today={ds === todayStr}
          >{day}</div>
        {/if}
      {/each}
    </div>
  </div>

  <!-- Weekly mileage chart -->
  <div class="card">
    <div class="card-header">
      <span class="card-label">weekly distance</span>
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

  <!-- Recent runs -->
  {#if recentRuns.length > 0}
  <div class="card">
    <div class="card-label">recent runs</div>
    <div class="runs-list">
      {#each recentRuns as run}
        <div class="run-row">
          <div class="run-date">{fmtDate(run.date)}</div>
          <div class="run-dist">{(run.distance * kmFactor).toFixed(1)} {unit}</div>
          {#if run.duration}<div class="run-dur">{run.duration}</div>{/if}
          <div class="run-notes">{run.notes || ''}</div>
          <button class="del-btn" onclick={() => deleteRun(run.id)}>×</button>
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
    margin-top: 6px;
  }
  .log-btn:hover { border-color: #555; }

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
  .submit-btn {
    background: #ff6b35;
    border: none;
    border-radius: 6px;
    color: #fff;
    font-size: 13px;
    font-weight: 500;
    padding: 8px 20px;
    cursor: pointer;
    letter-spacing: 0.04em;
  }
  .submit-btn:hover { background: #e55a28; }

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
    background: #222;
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
    color: #777;
    text-transform: uppercase;
    padding-bottom: 6px;
  }
  .cal-day {
    aspect-ratio: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 13px;
    color: #ccc;
    border-radius: 50%;
  }
  .cal-day.has-run {
    color: #ff6b35;
    font-weight: 600;
    background: rgba(255, 107, 53, 0.12);
  }
  .cal-day.is-today {
    color: #fff;
    border: 1px solid #333;
  }
  .cal-day.has-run.is-today {
    border-color: #ff6b35;
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
  /* ── Runs list ── */
  .runs-list {
    display: flex;
    flex-direction: column;
  }
  .run-row {
    display: flex;
    align-items: center;
    gap: 12px;
    padding: 9px 0;
    border-bottom: 1px solid #1c1c1c;
    font-size: 13px;
  }
  .run-row:last-child { border-bottom: none; }
  .run-date { color: #999; min-width: 52px; }
  .run-dist { color: #ff6b35; font-weight: 500; min-width: 50px; }
  .run-dur { color: #aaa; min-width: 44px; }
  .run-notes {
    color: #888;
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
</style>
