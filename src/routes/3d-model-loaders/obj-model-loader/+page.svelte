<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import { assets } from '$app/paths';

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    const light = new THREE.PointLight()
    light.position.set(2.5, 7.5, 15)
    scene.add(light)

    const camera = new THREE.PerspectiveCamera(
        75,
        rect.width / rect.height,
        0.1,
        1000
    )
    camera.position.z = 3

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    // const material = new THREE.MeshBasicMaterial({ color: 0x00ff00, wireframe: true })

    const objLoader = new OBJLoader()
    objLoader.load(
        `${assets}/cube.obj`,
        (object) => {
            /**
             * if you know the exact index of the mesh you want to change
             * you can do it like this
            */
            // (object.children[0] as THREE.Mesh).material = material

            /**
             * if you want to change all the meshes in the object
             * you can do it like this
            */
            // object.traverse(function (child) {
            //     if ((child as THREE.Mesh).isMesh) {
            //         (child as THREE.Mesh).material = material
            //     }
            // })
            scene.add(object)
        },
        (xhr) => {
            /**
             * this function executes when the object is loading
             * xhr.loaded returns the loaded bytes
             */
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
