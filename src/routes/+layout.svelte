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
            <li><a href="/maps/normalmap">Normal Map</a></li>
            <li><a href="/maps/displacement-map">Displacement Map</a></li>
            <li><a href="/maps/displacement-map-repeat-center">Displacement Map with Repeat and Center controls</a></li>
            <li><a href="/maps/texture-mipmap">Texture Mipmap</a></li>
            <li><a href="/maps/custom-mipmap">Custom Mipmap</a></li>
            <li><a href="/maps/mipmaps-with-anistropic-filtering">Mipmap with Anistropic Filtering</a></li>
          </ul>
        </li>

        <li>
          Lights
          <ul>
            <li><a href="/lights/ambient-light">Ambient Light</a></li>
            <li><a href="/lights/directional-light">Directional Light</a></li>
            <li><a href="/lights/hemisphere-light">Hemisphere Light</a></li>
            <li><a href="/lights/point-light">Point Light</a></li>
            <li><a href="/lights/spot-light">Spot Light</a></li>
          </ul>
        </li>

        <li>
          Shadows
          <ul>
            <li><a href="/lights/shadows/spot-light-shadow">Spot Light Shadow</a></li>
            <li><a href="/lights/shadows/directional-light-shadow">Directional Light Shadow</a></li>
          </ul>
        </li>

        <li>
          Controls
          <ul>
            <li><a href="/controls/orbit-controls">Orbit Controls</a></li>
            <li><a href="/controls/trackball-controls">Trackball Controls</a></li>
            <li><a href="/controls/pointer-lock-controls">Pointer Lock Controls</a></li>
            <li><a href="/controls/drag-controls">Drag Controls</a></li>
            <li><a href="/controls/transform-controls">Transform Controls</a></li>
            <li><a href="/controls/multiple-controls">Multiple Controls</a></li>
          </ul>
        </li>

        <li>
          3D Model Loaders
          <ul>
            <li><a href="/3d-model-loaders/obj-model-loader">OBJ Model Loader</a></li>
            <li><a href="/3d-model-loaders/mtl-model-loader">MTL Model Loader</a></li>
          </ul>
        </li>
      </ul>
    </nav>
  </header>  

  <main>
    <slot />
  </main>
</div>

<style lang="scss">
  .page-container {
    display: flex;
    flex-direction: row;
    width: 100vw;
    height: 100vh;
  }

  nav {
    width: 20rem;
    height: 100%;
    padding: 4rem 1rem 2rem;
    background-color: oklch(15% 0 0);
    overflow: auto;

    & > ul > li {
      margin-top: 1rem;
    }

    & li {
      margin-top: .25rem;
    }

    & ul {
      list-style: none;

      & ul {
        padding: .5rem 0 0 1rem;
      }

      & a {
        text-decoration: none;

        &:hover {
          text-decoration: underline;
          color:oklch(50% 30 120)
        }
      }
    }
  }

  main {
    width: calc(100vw - 20rem);
    height: 100%;
    background-color: oklch(0% 0 0);
  }
</style>