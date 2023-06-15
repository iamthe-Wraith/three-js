<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import { tweened } from 'svelte/motion';
  import * as SC from 'svelte-cubed';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import { assets } from '$app/paths';

  /**
   * https://sbcode.net/threejs/tween/#srctypingstweenjsindexdts
   * 
   * see also: https://sbcode.net/threejs/tween-animation-mixer/
  */

  const pos = tweened({ x: 0, y: 0, z: 0 }, { duration: 1000 })

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
    //renderer.physicallyCorrectLights = true //deprecated
    renderer.useLegacyLights = false //use this instead of setting physicallyCorrectLights=true property
    renderer.shadowMap.enabled = true
    renderer.outputEncoding = THREE.sRGBEncoding
    renderer.setSize(rect.width, rect.height)

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    const sceneMeshes: THREE.Mesh[] = []
    let monkey: THREE.Mesh

    const loader = new GLTFLoader()
    loader.load(
      `${assets}/monkey_textured.glb`,
      function (gltf) {
        gltf.scene.traverse(function (child) {
          if ((child as THREE.Mesh).isMesh) {
            const m = child as THREE.Mesh
            m.receiveShadow = true
            m.castShadow = true
            if (child.name === 'Plane') {
              sceneMeshes.push(m)
            } else if (child.name === 'Suzanne') {
              monkey = m
            }
          }
          if ((child as THREE.Light).isLight) {
            const l = child as THREE.Light
            l.castShadow = true
            l.shadow!.bias = -0.003
            l.shadow!.mapSize.width = 2048
            l.shadow!.mapSize.height = 2048
          }
        })
        scene.add(gltf.scene)
      },
      (xhr) => {
        console.log((xhr.loaded / xhr.total) * 100 + '% loaded')
      },
      (error) => {
        console.log(error)
      }
    )

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = rect.width / rect.height
      camera.updateProjectionMatrix()
      renderer.setSize(rect.width, rect.height)
      render()
    }

    const raycaster = new THREE.Raycaster()
    const mouse = new THREE.Vector2()

    let p: THREE.Vector3;

    function onDoubleClick(event: MouseEvent) {
      mouse.set(
        ((event.clientX - rect.left) / renderer.domElement.clientWidth) * 2 - 1,
        -(event.clientY / renderer.domElement.clientHeight) * 2 + 1
      )
      raycaster.setFromCamera(mouse, camera)

      const intersects = raycaster.intersectObjects(sceneMeshes, false)

      if (intersects.length > 0) {
        p = intersects[0].point

        $pos = { x: p.x, y: p.y, z: p.z };

        // new TWEEN.Tween(controls.target)
        //     .to({
        //         x: p.x,
        //         y: p.y,
        //         z: p.z
        //     }, 500)
        //     //.delay (1000)
        //     .easing(TWEEN.Easing.Cubic.Out)
        //     //.onUpdate(() => render())
        //     .start()
      }
    }
    renderer.domElement.addEventListener('dblclick', onDoubleClick, false)

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    function animate() {
      requestAnimationFrame(animate)

      controls.update()

      // TWEEN.update()

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

<div class="warning">getting tween.js to work in svelte wasn't quite working so I just moved on and have not gotten back around to figuring that out. might be better option to use <a href="https://svelte-cubed.vercel.app/">Svelte Cubed</a> instead.</div>

<style>
  .warning {
    position: absolute;
    top: 0;
    left: 0;
    background: red;
    color: white;
    padding: 1rem;
    font-size: 1.5rem;
    font-weight: bold;
    z-index: 100;
  }
</style>
