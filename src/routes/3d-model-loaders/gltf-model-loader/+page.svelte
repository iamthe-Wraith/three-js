<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import { assets } from '$app/paths';

  /**
   * !!!  UNOFFICIALLY THE PREFERRED FORMAT FOR THREEJS !!!
   * 
   * A loader for loading glTF models into the Threejs scene.
   * 
   * glTF is a specification for the efficient transmission and loading of 3D scenes and models.
   * 
   * glTF minimizes both the size of 3D assets, and the runtime processing needed to unpack and use those assets.
   * 
   * A glTF file may contain one or more scenes, meshes, materials, textures, skins, skeletons, morph targets, animations, lights and cameras.
   * 
   * Assets can be provided in either JSON (.gltf) or binary (.glb) format.
  */

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    // const light = new THREE.SpotLight()
    // light.position.set(5, 5, 5)
    // scene.add(light)

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    )
    camera.position.z = 2

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);
    // renderer.physicallyCorrectLights = true //deprecated
    renderer.useLegacyLights = false //use this instead of setting physicallyCorrectLights=true property
    renderer.shadowMap.enabled = true
    // renderer.outputEncoding = THREE.sRGBEncoding

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    const loader = new GLTFLoader()
    loader.load(
      `${assets}/monkey.glb`,
      function (gltf) {
        gltf.scene.traverse(function (child) {
          if ((child as THREE.Mesh).isMesh) {
            const m = (child as THREE.Mesh)
            m.receiveShadow = true
            m.castShadow = true
          }
          if (((child as THREE.Light)).isLight) {
            const l = (child as THREE.Light)
            l.castShadow = true
            l.shadow!.bias = -.003
            l.shadow!.mapSize.width = 2048
            l.shadow!.mapSize.height = 2048
          }
        })
        console.log(gltf.scene);
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
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
      render()
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

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
