<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls';
  import Stats from 'three/examples/jsm/libs/stats.module';

  /**
   * TrackballControls is similar to the OrbitControls. However, it does not maintain a constant 
   * camera up vector. That means that the camera can orbit past its polar extremes. It won't flip 
   * to stay the right side up.
  */

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    const camera = new THREE.PerspectiveCamera(
        75,
        rect.width / rect.height,
        0.1,
        1000
    )
    camera.position.z = 2

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);

    const controls = new TrackballControls(camera, renderer.domElement)
    // controls.addEventListener('change', () => console.log("Controls Change"))
    // controls.addEventListener('start', () => console.log("Controls Start Event"))
    // controls.addEventListener('end', () => console.log("Controls End Event"))
    // controls.enabled = false
    // controls.rotateSpeed = 1.0
    // controls.zoomSpeed = 1.2
    // controls.panSpeed = 0.8
    // controls.keys = ['KeyA', 'KeyS', 'KeyD']
    // controls.noPan = true //default false
    // controls.noRotate = true //default false
    // controls.noZoom = true //default false
    // controls.staticMoving = true //default false
    // controls.dynamicDampingFactor = 0.1
    // controls.maxDistance = 4
    // controls.minDistance = 2

    const geometry = new THREE.BoxGeometry()
    const material = new THREE.MeshBasicMaterial({
      color: 0x00ff00,
      wireframe: true,
    })

    const cube = new THREE.Mesh(geometry, material)
    scene.add(cube)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
      render()
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    function animate() {
      requestAnimationFrame(animate)

      // trackball controls needs to be updated in the animation loop before it will work
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
