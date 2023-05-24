<script lang="ts" type="module">
    import { onMount } from 'svelte';
    import * as THREE from 'three'
    import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
    import Stats from 'three/examples/jsm/libs/stats.module'

  onMount(() => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.background = new THREE.Color(0x111111);

    const camera1 = new THREE.PerspectiveCamera(
      75,
      rect.width / (rect.height / 2),
      0.1,
      1000
    );

    const asp = (rect.height / 2) / rect.width;
    console.log('asp: ', asp);
    const camera2 = new THREE.OrthographicCamera(
      -2 - asp, 
      2 + asp, 
      -2, 
      2, 
      1, 
      1000
    );

    camera1.position.z = 2;
    camera2.position.z = 2;

    /**
     * WebGLRenderer is the most used, fastest, and most supported renderer.
     */
    const renderer1 = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer1.setSize(rect.width, (rect.height / 2));

    const renderer2 = new THREE.WebGLRenderer({ canvas: document.querySelector('#c2') as HTMLCanvasElement })
    renderer2.setSize(rect.width, (rect.height / 2));

    const controls = new OrbitControls( camera1, renderer1.domElement );

    const geometry = new THREE.BoxGeometry()
    const material = new THREE.MeshBasicMaterial({
      color: 0x00ff00,
      wireframe: true,
    })

    const cube = new THREE.Mesh(geometry, material)
    scene.add(cube)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera1.aspect = rect.width / (rect.height / 2);
      camera1.updateProjectionMatrix();

      renderer1.setSize(rect.width, (rect.height / 2));
      renderer2.setSize(rect.width, (rect.height / 2));
      
      render();
    }

    function animate() {
      requestAnimationFrame(animate)

      cube.rotation.x += 0.01
      cube.rotation.y += 0.01

      render()
      stats.update();
    }

    const stats = new Stats();
    main.appendChild(stats.dom);

    function render() {
      renderer1.render(scene, camera1)
      renderer2.render(scene, camera2)
    }

    animate()
  })
</script>

<canvas id="c1"></canvas>
<canvas id="c2"></canvas>
