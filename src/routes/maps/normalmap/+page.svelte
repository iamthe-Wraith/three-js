<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import worldColorFile from '$assets/specularMaps/worldColour.5400x2700.jpg';
  import normalMapFile from '$assets/bumpMaps/earth_normalmap_8192x4096.jpg';

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

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

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    const planeGeometry = new THREE.PlaneGeometry(3.6, 1.8)

    const material = new THREE.MeshPhongMaterial()

    const texture = new THREE.TextureLoader().load(worldColorFile);
    material.map = texture

    /**
     * Normal maps are better than bump maps.
     */
    const normalTexture = new THREE.TextureLoader().load(normalMapFile);
    material.normalMap = normalTexture
    material.normalScale.set(2, 2)

    const plane = new THREE.Mesh(planeGeometry, material)
    scene.add(plane)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
      render()
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    const gui = new (await import('dat.gui')).GUI();

    gui.add(material.normalScale, 'x', 0, 10, 0.01)
    gui.add(material.normalScale, 'y', 0, 10, 0.01)
    gui.add(light.position, 'x', -20, 20).name('Light Pos X')

    function animate() {
      requestAnimationFrame(animate)

      controls.update()

      render()

      stats.update()
    }

    function render() {
      renderer.render(scene, camera)
    }

    animate()
  })
</script>

<canvas id="c1"></canvas>
