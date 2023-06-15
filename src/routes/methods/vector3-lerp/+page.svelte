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
    camera.position.set(1, 2, 5)

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height)

    const controls = new OrbitControls(camera, renderer.domElement)

    const floor = new THREE.Mesh(
      new THREE.PlaneGeometry(20, 20, 10, 10),
      new THREE.MeshBasicMaterial({ color: 0xaec6cf, wireframe: true })
    )
    floor.rotateX(-Math.PI / 2)
    scene.add(floor)

    const geometry = new THREE.BoxGeometry()
    //the cube used for .lerp
    const cube1 = new THREE.Mesh(
      geometry,
      new THREE.MeshBasicMaterial({ color: 0x00ff00, wireframe: true })
    )
    cube1.position.y = 0.5
    scene.add(cube1)

    //the cube used for .lerpVectors
    const cube2 = new THREE.Mesh(
      geometry,
      new THREE.MeshBasicMaterial({ color: 0xff0000, wireframe: true })
    )
    cube2.position.y = 0.5
    scene.add(cube2)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = rect.width / rect.height
      camera.updateProjectionMatrix()
      renderer.setSize(rect.width, rect.height)
      render()
    }

    const raycaster = new THREE.Raycaster()
    let v1 = new THREE.Vector3(2, 0.5, 2)
    let v2 = new THREE.Vector3(0, 0.5, 0)
    const mouse = new THREE.Vector2()

    function onDoubleClick(event: THREE.Event) {
      mouse.set(
        ((event.clientX - rect.left) / renderer.domElement.clientWidth) * 2 - 1,
        -(event.clientY / renderer.domElement.clientHeight) * 2 + 1
      )
      raycaster.setFromCamera(mouse, camera)

      const intersects = raycaster.intersectObject(floor, false)
      if (intersects.length > 0) {
        v1 = intersects[0].point
        v1.y += 0.5 //raise it so it appears to sit on grid
        //console.log(v1)
      }
    }
    renderer.domElement.addEventListener('dblclick', onDoubleClick, false)

    const stats = new Stats()
    document.body.appendChild(stats.dom)
    const data = {
      lerpAlpha: 0.1,
      lerpVectorsAlpha: 1.0,
    }
    const gui = new (await import('dat.gui')).GUI();

    const lerpFolder = gui.addFolder('.lerp')
    lerpFolder.add(data, 'lerpAlpha', 0, 1.0, 0.01)
    lerpFolder.open()

    const lerpVectorsFolder = gui.addFolder('.lerpVectors')
    lerpVectorsFolder.add(data, 'lerpVectorsAlpha', 0, 1.0, 0.01)
    lerpVectorsFolder.open()

    function animate() {
      requestAnimationFrame(animate)
      controls.update()

      /**
       * move cube1 to a new position (v1) at a particular distance
       * per frame (data.lerpAlpha)
       */
      cube1.position.lerp(v1, data.lerpAlpha)

      /**
       * get a point between 2 vectors
      */
      cube2.position.lerpVectors(v1, v2, data.lerpVectorsAlpha)
      controls.target.copy(cube1.position)
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
