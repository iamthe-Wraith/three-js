<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import gridFile from '$assets/grid.png';

  /**
   * mipmaps allow threejs to create smaller versions of a texture (in the background)
   * and use them when the texture is far away from the camera.
   * 
   * this greatly improves performance
   * 
   * in this example, there is a split screen where threejs renders the default
   * mipmaps on the left, and the custom mipmaps on the right.
  */

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    /**
     * notice 2 scenes here. each section of the split screen is a
     * different scene.
     */
    const scene1 = new THREE.Scene()
    const scene2 = new THREE.Scene()

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    const light = new THREE.PointLight(0xffffff, 2)
    light.position.set(0, 2, 5)
    scene.add(light)

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    );

    camera.position.z = 1;

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);

    new OrbitControls(camera, renderer.domElement)

    const planeGeometry1 = new THREE.PlaneGeometry()
    const planeGeometry2 = new THREE.PlaneGeometry()

    const texture1 = new THREE.TextureLoader().load(gridFile);
    const texture2 = texture1.clone()

    const material1 = new THREE.MeshBasicMaterial({ map: texture1 })
    const material2 = new THREE.MeshBasicMaterial({ map: texture2 })

    texture2.minFilter = THREE.NearestFilter
    texture2.magFilter = THREE.NearestFilter

    const plane1 = new THREE.Mesh(planeGeometry1, material1)
    const plane2 = new THREE.Mesh(planeGeometry2, material2)

    scene1.add(plane1)
    scene2.add(plane2)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
        camera.aspect = window.innerWidth / window.innerHeight
        camera.updateProjectionMatrix()
        renderer.setSize(window.innerWidth, window.innerHeight)
        render()
    }

    const options = {
        minFilters: {
            NearestFilter: THREE.NearestFilter,
            NearestMipMapLinearFilter: THREE.NearestMipMapLinearFilter,
            NearestMipMapNearestFilter: THREE.NearestMipMapNearestFilter,
            'LinearFilter ': THREE.LinearFilter,
            'LinearMipMapLinearFilter (Default)': THREE.LinearMipMapLinearFilter,
            LinearMipmapNearestFilter: THREE.LinearMipmapNearestFilter,
        },
        magFilters: {
            NearestFilter: THREE.NearestFilter,
            'LinearFilter (Default)': THREE.LinearFilter,
        },
    }
    const gui = new (await import('dat.gui')).GUI();

    const textureFolder = gui.addFolder('THREE.Texture')
    textureFolder
      .add(texture2, 'minFilter', options.minFilters)
      .onChange(() => updateMinFilter())
    textureFolder
      .add(texture2, 'magFilter', options.magFilters)
      .onChange(() => updateMagFilter())
    textureFolder.open()

    function updateMinFilter() {
      texture2.minFilter = Number(texture2.minFilter) as THREE.TextureFilter
      texture2.needsUpdate = true
    }
    function updateMagFilter() {
      texture2.magFilter = Number(texture2.magFilter) as THREE.MagnificationTextureFilter
      texture2.needsUpdate = true
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    function animate() {
      requestAnimationFrame(animate)

      render()

      stats.update()
    }

    function render() {
      /**
       * notice the scissor test here. this is what allows us to split the screen
      */
      renderer.setScissorTest(true)
      renderer.setScissor(0, 0, rect.width / 2 - 2, rect.height)
      renderer.render(scene1, camera)

      renderer.setScissor(
        rect.width / 2,
        0,
        rect.width / 2 - 2,
        rect.height
      )
      renderer.render(scene2, camera)

      renderer.setScissorTest(false)
    }
    animate()
  })
</script>

<canvas id="c1"></canvas>
