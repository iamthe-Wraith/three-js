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
   * with anistropic filtering, the mipmaps are created differently. instead of the
   * the new mipmaps being a smaller version of the previous mipmap, the new mipmaps
   * are created a more vertical set by cutting the width in half and keeping the height,
   * then cutting the height in half and keeping the width, and so on. see
   * 
   * https://www.udemy.com/course/threejs-tutorials/learn/lecture/18485580
   * at 0 min:26 sec
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

    const planeGeometry1 = new THREE.PlaneGeometry(2, 25)
    const planeGeometry2 = new THREE.PlaneGeometry(2, 25)

    // const texture1 = new THREE.TextureLoader().load("img/grid.png")
    // const texture2 = new THREE.TextureLoader().load("img/grid.png")

    const mipmap = (size: number, color: string): HTMLCanvasElement => {
      const imageCanvas = document.createElement('canvas') as HTMLCanvasElement
      const context = imageCanvas.getContext('2d') as CanvasRenderingContext2D
      imageCanvas.width = size
      imageCanvas.height = size
      context.fillStyle = '#888888'
      context.fillRect(0, 0, size, size)
      context.fillStyle = color
      context.fillRect(0, 0, size / 2, size / 2)
      context.fillRect(size / 2, size / 2, size / 2, size / 2)
      return imageCanvas
    }

    const blankCanvas = document.createElement('canvas') as HTMLCanvasElement
    blankCanvas.width = 128
    blankCanvas.height = 128

    const texture1 = new THREE.CanvasTexture(blankCanvas)
    texture1.mipmaps[0] = mipmap(128, '#ff0000')
    texture1.mipmaps[1] = mipmap(64, '#00ff00')
    texture1.mipmaps[2] = mipmap(32, '#0000ff')
    texture1.mipmaps[3] = mipmap(16, '#880000')
    texture1.mipmaps[4] = mipmap(8, '#008800')
    texture1.mipmaps[5] = mipmap(4, '#000088')
    texture1.mipmaps[6] = mipmap(2, '#008888')
    texture1.mipmaps[7] = mipmap(1, '#880088')
    texture1.repeat.set(5, 50)
    texture1.wrapS = THREE.RepeatWrapping
    texture1.wrapT = THREE.RepeatWrapping

    const texture2 = texture1.clone()
    texture2.minFilter = THREE.NearestFilter
    texture2.magFilter = THREE.NearestFilter

    const material1 = new THREE.MeshBasicMaterial({ map: texture1 })
    const material2 = new THREE.MeshBasicMaterial({ map: texture2 })

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
    textureFolder
      .add(texture2, 'anisotropy', 1, renderer.capabilities.getMaxAnisotropy())
      .onChange(() => updateAnistropy())
    textureFolder.open()

    function updateAnistropy() {
      material2.map = texture2.clone()
    }
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
      renderer.setScissorTest(true)

      renderer.setScissor(0, 0, window.innerWidth / 2 - 2, window.innerHeight)
      renderer.render(scene1, camera)

      renderer.setScissor(
        window.innerWidth / 2,
        0,
        window.innerWidth / 2 - 2,
        window.innerHeight
      )
      renderer.render(scene2, camera)

      renderer.setScissorTest(false)
    }
    animate()
  })
</script>

<canvas id="c1"></canvas>
