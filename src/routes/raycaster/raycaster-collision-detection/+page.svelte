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

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    )
    camera.position.z = 2

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height)

    const raycaster = new THREE.Raycaster()
    const sceneMeshes: THREE.Mesh[] = []
    const dir = new THREE.Vector3()
    let intersects: THREE.Intersection[] = []

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.addEventListener('change', function () {
      /**
       * when the orbit controls change (ie. when you look, move, or pan around),
       * we want to move the reticle to the center of the screen, and move the
       * lines to the camera position.
      */
      xLine.position.copy(controls.target)
      yLine.position.copy(controls.target)
      zLine.position.copy(controls.target)

      raycaster.set(
        controls.target,
        dir.subVectors(camera.position, controls.target).normalize()
      )

      intersects = raycaster.intersectObjects(sceneMeshes, false)
      if (intersects.length > 0 && intersects[0].distance < controls.target.distanceTo(camera.position)) {
        camera.position.copy(intersects[0].point)
      }
    })

    const floor = new THREE.Mesh(
      new THREE.PlaneGeometry(10, 10),
      new THREE.MeshNormalMaterial({ side: THREE.DoubleSide })
    )
    floor.rotateX(-Math.PI / 2)
    floor.position.y = -1
    scene.add(floor)
    sceneMeshes.push(floor)

    const wall1 = new THREE.Mesh(
      new THREE.PlaneGeometry(2, 2),
      new THREE.MeshNormalMaterial({ side: THREE.DoubleSide })
    )
    wall1.position.x = 4
    wall1.rotateY(-Math.PI / 2)
    scene.add(wall1)
    sceneMeshes.push(wall1)

    const wall2 = new THREE.Mesh(
      new THREE.PlaneGeometry(2, 2),
      new THREE.MeshNormalMaterial({ side: THREE.DoubleSide })
    )
    wall2.position.z = -3
    scene.add(wall2)
    sceneMeshes.push(wall2)

    const cube: THREE.Mesh = new THREE.Mesh(
      new THREE.BoxGeometry(),
      new THREE.MeshNormalMaterial()
    )
    cube.position.set(-3, 0, 0)
    scene.add(cube)
    sceneMeshes.push(cube)

    const ceiling = new THREE.Mesh(
      new THREE.PlaneGeometry(10, 10),
      new THREE.MeshNormalMaterial({ side: THREE.DoubleSide })
    )
    ceiling.rotateX(Math.PI / 2)
    ceiling.position.y = 3
    scene.add(ceiling)
    sceneMeshes.push(ceiling)

    //crosshair
    const lineMaterial = new THREE.LineBasicMaterial({
      color: 0x0000ff,
    })
    const points: THREE.Vector3[] = []
    points[0] = new THREE.Vector3(-0.1, 0, 0)
    points[1] = new THREE.Vector3(0.1, 0, 0)
    let lineGeometry = new THREE.BufferGeometry().setFromPoints(points)
    const xLine = new THREE.Line(lineGeometry, lineMaterial)
    scene.add(xLine)
    points[0] = new THREE.Vector3(0, -0.1, 0)
    points[1] = new THREE.Vector3(0, 0.1, 0)
    lineGeometry = new THREE.BufferGeometry().setFromPoints(points)
    const yLine = new THREE.Line(lineGeometry, lineMaterial)
    scene.add(yLine)
    points[0] = new THREE.Vector3(0, 0, -0.1)
    points[1] = new THREE.Vector3(0, 0, 0.1)
    lineGeometry = new THREE.BufferGeometry().setFromPoints(points)
    const zLine = new THREE.Line(lineGeometry, lineMaterial)
    scene.add(zLine)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = rect.width / rect.height
      camera.updateProjectionMatrix()
      renderer.setSize(rect.width, rect.height)
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
