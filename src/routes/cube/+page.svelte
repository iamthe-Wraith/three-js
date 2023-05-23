<script lang="ts">
    import { onMount } from 'svelte';
    import * as THREE from 'three'

  onMount(() => {
    const scene = new THREE.Scene()

    const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
    )
    camera.position.z = 2

    const main = document.querySelector('main')!;
    const rect = main.getBoundingClientRect();

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c')! })
    // renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setSize(rect.width, rect.height);

    camera.aspect = rect.width / rect.height;
    camera.updateProjectionMatrix();

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

    function animate() {
        requestAnimationFrame(animate)

        cube.rotation.x += 0.01
        cube.rotation.y += 0.01

        render()
    }

    function render() {
        renderer.render(scene, camera)
    }

    animate()
  })
</script>

<canvas id="c"></canvas>
