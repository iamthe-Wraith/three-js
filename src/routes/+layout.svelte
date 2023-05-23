<script lang="ts">
  import './global.css';
</script>

<div class=page-container>
  <header>
    <nav>
      <ul>
        <li><a href="/cube">Cube</a></li>
      </ul>
    </nav>
  </header>  

  <main>
    <slot />
  </main>
</div>

<style>
  .page-container {
    display: flex;
    flex-direction: row;
    width: 100vw;
    height: 100vh;
  }

  nav {
    width: 20rem;
    height: 100%;
    padding: 2rem 1rem;
    background-color: oklch(15% 0 0);

    & ul {
      list-style: none;

      & a {
        text-decoration: none;
      }
    }
  }

  main {
    width: calc(100vw - 20rem);
    height: 100%;
    background-color: oklch(0% 0 0);
  }
</style>