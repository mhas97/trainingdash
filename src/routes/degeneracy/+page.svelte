<script>
  let habits = $state(
    JSON.parse(localStorage.getItem('habits') ?? 'null') ?? [
      { name: 'alcohol', since: '2026-05-04' },
      { name: 'vape', since: '2026-05-05' },
      { name: 'porn', since: '2026-04-01' },
      { name: 'drugs', since: '2025-10-11' },
    ]
  );

  const accents = ['#b44fff', '#00e5ff', '#ff4f7b', '#ffb800'];

  function daysSince(dateStr) {
    const diff = Date.now() - new Date(dateStr).getTime();
    return Math.floor(diff / (1000 * 60 * 60 * 24));
  }

  function brokeStreak(index) {
    habits[index].since = new Date().toISOString().split('T')[0];
  }

  $effect(() => {
    localStorage.setItem('habits', JSON.stringify(habits));
  });
</script>

<div class="page">
  <h1>degeneracy</h1>
  <p class="subtitle">days since last slip-up</p>

  <div class="grid">
    {#each habits as habit, i}
      <div class="card" style="--accent: {accents[i % accents.length]}">
        <div class="name">{habit.name}</div>
        <div class="days">{daysSince(habit.since)}</div>
        <div class="unit">days clean</div>
        <button class="broke-btn" onclick={() => brokeStreak(i)}>broke streak</button>
      </div>
    {/each}
  </div>
</div>

<style>
  .page {
    background: #0d0d0f;
    min-height: 100vh;
    padding: 2rem;
    font-family: inherit;
  }
  h1 {
    font-size: 40px;
    font-weight: 500;
    color: #fff;
    margin-bottom: 4px;
  }
  .subtitle {
    font-size: 15px;
    color: #555;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: 1.75rem;
  }
  .grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }
  .card {
    background: #131318;
    border: 1px solid #222;
    border-radius: 12px;
    padding: 1.1rem 1.25rem;
    border-top: 2px solid var(--accent);
  }
  .name {
    font-size: 20px;
    color: #555;
    text-transform: uppercase;
    letter-spacing: 0.08em;
    margin-bottom: 10px;
  }
  .days {
    font-size: 40px;
    font-weight: 500;
    line-height: 1;
    color: var(--accent);
    margin-bottom: 2px;
  }
  .unit {
    font-size: 16px;
    color: #444;
    margin-bottom: 14px;
  }
</style>