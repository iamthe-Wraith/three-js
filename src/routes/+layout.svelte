<script lang="ts">
  import './global.css';
</script>

<div class=page-container>
  <header>
    <nav>
      <ul>
        <li>Examples</li>
        <ul>
          <li><a href="/examples/mesh-basic-material">Mesh Basic Material</a></li>
          <li><a href="/examples/mesh-normal-material">Mesh Normal Material</a></li>
          <li><a href="/examples/mesh-lambert-material">Mesh Lambert Material</a></li>
        </ul>
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

      & li:has(+ ul) {
        font-size: 1.1rem;
        font-weight: bold;
      }

      & ul {
        padding: .5rem 0 0 1rem;
      }

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