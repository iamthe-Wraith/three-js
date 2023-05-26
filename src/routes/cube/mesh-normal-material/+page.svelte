<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module'
  // import { GUI } from 'dat.gui';

onMount(async () => {
  const main = document.querySelector('main') as HTMLCanvasElement;
  const rect = main.getBoundingClientRect();

  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x111111);
  scene.add(new THREE.AxesHelper(5));

  const camera1 = new THREE.PerspectiveCamera(
    75,
    rect.width / rect.height,
    0.1,
    1000
  );

  camera1.position.z = 6.5;
  camera1.position.y = 2.5;
  camera1.position.x = 2.5;

  /**
   * WebGLRenderer is the most used, fastest, and most supported renderer.
   */
  const renderer1 = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
  renderer1.setSize(rect.width, rect.height);

  const controls = new OrbitControls( camera1, renderer1.domElement );

  const geometry = new THREE.BoxGeometry()
  const material = new THREE.MeshNormalMaterial({
    transparent: true,
    opacity: 0.5,
  })

  const cube1 = new THREE.Mesh(geometry, material)
  const cube2 = new THREE.Mesh(geometry, material)
  const cube3 = new THREE.Mesh(geometry, material)
  const cube4 = new THREE.Mesh(geometry, material)
  const cube5 = new THREE.Mesh(geometry, material)

  scene.add(cube1)
  scene.add(cube2)
  scene.add(cube3)
  scene.add(cube4)
  scene.add(cube5)

  cube1.position.x = -2.25
  cube2.position.x = -0.75
  cube3.position.x = 0.75
  cube4.position.x = 2.25
  cube5.position.x = 3.75

  window.addEventListener('resize', onWindowResize, false)
  function onWindowResize() {
    camera1.aspect = rect.width / rect.height;
    camera1.updateProjectionMatrix();

    renderer1.setSize(rect.width, rect.height);
    
    render();
  }

  // const gui = new GUI();
  const gui = new (await import('dat.gui')).GUI();

  const cubeFolder = gui.addFolder('Cube');
  cubeFolder.open();

  const cubeRotationFolder = cubeFolder.addFolder('Rotation');
  cubeRotationFolder.add(cube1.rotation, 'x', 0, Math.PI * 2);
  cubeRotationFolder.add(cube1.rotation, 'y', 0, Math.PI * 2);
  cubeRotationFolder.add(cube1.rotation, 'z', 0, Math.PI * 2);

  const cubeMeshFolder = cubeFolder.addFolder('Material');
  cubeMeshFolder.open();
  cubeMeshFolder.add(material, 'wireframe');
  cubeMeshFolder.add(material, 'transparent');
  cubeMeshFolder.add(material, 'opacity', 0, 1, 0.01);
  cubeMeshFolder.add(material, 'depthTest');
  cubeMeshFolder.add(material, 'depthWrite');
  cubeMeshFolder.add(material, 'visible');
  cubeMeshFolder.add(material, 'side', {
    FrontSide: THREE.FrontSide,
    BackSide: THREE.BackSide,
    DoubleSide: THREE.DoubleSide,
  }).onChange(() => {
    material.side = Number(material.side) as THREE.Side;
    material.needsUpdate = true
  });

  const cameraFolder = gui.addFolder('Camera');

  const cameraPositionFolder = cameraFolder.addFolder('Position');
  cameraPositionFolder.add(camera1.position, 'x', -10, 10);
  cameraPositionFolder.add(camera1.position, 'y', -10, 10);
  cameraPositionFolder.add(camera1.position, 'z', -10, 10);

  const cameraRotationFolder = cameraFolder.addFolder('Rotation');
  cameraRotationFolder.add(camera1.rotation, 'x', 0, Math.PI * 2);
  cameraRotationFolder.add(camera1.rotation, 'y', 0, Math.PI * 2);
  cameraRotationFolder.add(camera1.rotation, 'z', 0, Math.PI * 2);

  function animate() {
    requestAnimationFrame(animate)

    // cube.rotation.x += 0.01
    // cube.rotation.y += 0.01

    render()
    stats.update();
  }

  const stats = new Stats();
  main.appendChild(stats.dom);

  function render() {
    renderer1.render(scene, camera1)
  }

  animate()
})
</script>

<canvas id="c1"></canvas>
