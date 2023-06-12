<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { PointerLockControls } from 'three/examples/jsm/controls/PointerLockControls';
  import Stats from 'three/examples/jsm/libs/stats.module';

  /**
   * The PointerLockControls implements the inbuilt browsers Pointer Lock API. It provides input methods 
   * based on the movement of the mouse over time (i.e., deltas), not just the absolute position of the 
   * mouse cursor in the viewport. It gives you access to raw mouse movement, locks the target of mouse 
   * events to a single element, eliminates limits on how far mouse movement can go in a single direction, 
   * and removes the cursor from view. It is ideal for first person 3D games, for example.
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
    camera.position.y = 1
    camera.position.z = 2

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);

    const menuPanel = document.getElementById('menuPanel') as HTMLDivElement
    const startButton = document.getElementById('startButton') as HTMLInputElement
    startButton.addEventListener(
      'click',
      function () {
        controls.lock()
      },
      false
    )

    const controls = new PointerLockControls(camera, renderer.domElement)
    // controls.addEventListener('change', () => console.log("Controls Change"))
    controls.addEventListener('lock', () => menuPanel.style.display = 'none')
    controls.addEventListener('unlock', () => menuPanel.style.display = 'block')

    const planeGeometry = new THREE.PlaneGeometry(100, 100, 50, 50)
    const material = new THREE.MeshBasicMaterial({
      color: 0x00ff00,
      wireframe: true,
    })
    const plane = new THREE.Mesh(planeGeometry, material)
    plane.rotateX(-Math.PI / 2)
    scene.add(plane)

    const cubes: THREE.Mesh[] = []
    for (let i = 0; i < 100; i++) {
      const geo = new THREE.BoxGeometry(
        Math.random() * 4,
        Math.random() * 16,
        Math.random() * 4
      )
      const mat = new THREE.MeshBasicMaterial({ wireframe: true })
      switch (i % 3) {
        case 0:
          mat.color = new THREE.Color(0xff0000)
          break
        case 1:
          mat.color = new THREE.Color(0xffff00)
          break
        case 2:
          mat.color = new THREE.Color(0x0000ff)
          break
      }
      const cube = new THREE.Mesh(geo, mat)
      cubes.push(cube)
    }
    cubes.forEach((c) => {
      c.position.x = Math.random() * 100 - 50
      c.position.z = Math.random() * 100 - 50
      c.geometry.computeBoundingBox()
      c.position.y =
        ((c.geometry.boundingBox as THREE.Box3).max.y -
          (c.geometry.boundingBox as THREE.Box3).min.y) /
        2
      scene.add(c)
    })

    const onKeyDown = function (event: KeyboardEvent) {
      switch (event.code) {
        case "KeyW":
          controls.moveForward(.25)
          break
        case "KeyA":
          controls.moveRight(-.25)
          break
        case "KeyS":
          controls.moveForward(-.25)
          break
        case "KeyD":
          controls.moveRight(.25)
          break
      }
    }
    document.addEventListener('keydown', onKeyDown, false)

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

      //controls.update()

      render()

      stats.update()
    }

    function render() {
      renderer.render(scene, camera)
    }

    animate()
  })
</script>

<div id="menuPanel">
  <button id="startButton">Click to Start</button>
</div>

<canvas id="c1"></canvas>

<style>
  #menuPanel {
    position: absolute;
    background-color: rgba(255, 255, 255, 0.5);
    top: 0px;
    left: 0px;
    width: 100%;
    height: 100%;
  }

  #startButton {
    height: 50px;
    width: 200px;
    margin: -25px -100px;
    position: relative;
    top: 50%;
    left: 50%;
    font-size: 32px;
    color: black
  }
</style>
