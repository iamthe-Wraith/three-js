<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import { assets } from '$app/paths';

  /**
   * This is a simple raycaster example that shows how to use raycaster to detect collisions.
   * 
   * if you use the orbit controls to look, move, and pan around, if an object gets between
   * "you" and the reticle, the camera will move to the point where the object that got in
   * the way is.
  */

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    const light = new THREE.SpotLight()
    light.position.set(12.5, 12.5, 12.5)
    light.castShadow = true
    light.shadow.mapSize.width = 1024
    light.shadow.mapSize.height = 1024
    scene.add(light)

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    )
    camera.position.set(15, 15, 15)

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.shadowMap.enabled = true
    renderer.outputEncoding = THREE.sRGBEncoding
    renderer.setSize(rect.width, rect.height)

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    const pickableObjects: THREE.Mesh[] = []
    let intersectedObject: THREE.Object3D | null
    const originalMaterials: { [id: string]: THREE.Material | THREE.Material[] } =
      {}
    const highlightedMaterial = new THREE.MeshBasicMaterial({
      wireframe: true,
      color: 0x00ff00,
    })

    const loader = new GLTFLoader()
    loader.load(
      `${assets}/simple-scene.glb`,
      function (gltf) {
        gltf.scene.traverse(function (child) {
          if ((child as THREE.Mesh).isMesh) {
            const m = child as THREE.Mesh
            //the sphere and plane will not be mouse picked. THe plane will receive shadows while everything else casts shadows.
            switch (m.name) {
              case 'Plane':
                m.receiveShadow = true
                break
              case 'Sphere':
                m.castShadow = true
                break
              default:
                m.castShadow = true
                pickableObjects.push(m)
                //store reference to original materials for later
                originalMaterials[m.name] = (m as THREE.Mesh).material
            }
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
    let intersects: THREE.Intersection[]

    const mouse = new THREE.Vector2()

    function onDocumentMouseMove(event: MouseEvent) {
      mouse.set(
        ((event.clientX - rect.left) / renderer.domElement.clientWidth) * 2 - 1,
        -(event.clientY / renderer.domElement.clientHeight) * 2 + 1
      )
      raycaster.setFromCamera(mouse, camera)
      intersects = raycaster.intersectObjects(pickableObjects, false)

      if (intersects.length > 0) {
        intersectedObject = intersects[0].object
      } else {
        intersectedObject = null
      }
      pickableObjects.forEach((o: THREE.Mesh, i) => {
        if (intersectedObject && intersectedObject.name === o.name) {
          pickableObjects[i].material = highlightedMaterial
        } else {
          pickableObjects[i].material = originalMaterials[o.name]
        }
      })
    }
    document.addEventListener('mousemove', onDocumentMouseMove, false)

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
