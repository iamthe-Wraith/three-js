<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { TransformControls } from 'three/examples/jsm/controls/TransformControls';
  import Stats from 'three/examples/jsm/libs/stats.module';

  /**
   * Allows you to change the transforms of an object within the scene.
   * 
   * You attach the controls to the object, and then add the controls to the scene, so that the interaction handles are visible.
   * 
   * And then you can rescale, rotate and position the object within the scene.
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

    const geometry = new THREE.BoxGeometry()
    const material = new THREE.MeshNormalMaterial()

    const cube = new THREE.Mesh(geometry, material)
    scene.add(cube)

    const controls = new TransformControls(camera, renderer.domElement)
    controls.attach(cube)
    scene.add(controls)

    window.addEventListener('keydown', function (event) {
      switch (event.code) {
        case 'KeyG':
          controls.setMode('translate')
          break
        case 'KeyR':
          controls.setMode('rotate')
          break
        case 'KeyS':
          controls.setMode('scale')
          break
      }
    })

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

      // controls.update()

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
