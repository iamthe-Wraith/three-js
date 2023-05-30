<script lang="ts">
  import './global.css';
</script>

<div class=page-container>
  <header>
    <nav>
      <ul>
        <li>
          Materials
          <ul>
            <li><a href="/materials/mesh-basic-material">Mesh Basic Material</a></li>
            <li><a href="/materials/mesh-normal-material">Mesh Normal Material</a></li>
            <li><a href="/materials/mesh-lambert-material">Mesh Lambert Material</a></li>
            <li><a href="/materials/mesh-phong-material">Mesh Phong Material</a></li>
            <li><a href="/materials/mesh-standard-material">Mesh Standard Material</a></li>
            <li><a href="/materials/mesh-physical-material">Mesh Physical Material</a></li>
            <li><a href="/materials/mesh-matcap-material">Mesh Matcap Material</a></li>
            <li><a href="/materials/mesh-toon-material">Mesh Toon Material</a></li>
          </ul>
        </li>
        <li>
          Maps
          <ul>
            <li><a href="/maps/specular-map">Specular Map</a></li>
            <li><a href="/maps/roughness-metalness-maps">Roughness and Metalness Maps</a></li>
            <li><a href="/maps/bumpmap">Bump Map</a></li>
          </ul>
        </li>
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