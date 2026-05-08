<script>
  import '../app.css';
  import { page } from '$app/stores';

  let { children } = $props();
</script>

<nav>
  <a href="/" class:active={$page.url.pathname === '/'}>runs</a>
  <a href="/journal" class:active={$page.url.pathname === '/journal'}>journal</a>
  <a href="/degeneracy" class:active={$page.url.pathname === '/degeneracy'}>degeneracy</a>
</nav>

<main>
  {@render children()}
</main>

<style>
  nav {
    display: flex;
    gap: 1rem;
    padding: 1rem;
    border-bottom: 1px solid #eee;
  }

  nav a {
    text-decoration: none;
    color: #888;
    font-size: 14px;
  }

  nav a.active {
    color: black;
    font-weight: 500;
  }
</style>