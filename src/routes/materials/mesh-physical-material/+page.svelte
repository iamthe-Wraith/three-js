<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module'

onMount(async () => {
  const main = document.querySelector('main') as HTMLCanvasElement;
  const rect = main.getBoundingClientRect();

  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x111111);
  scene.add(new THREE.AxesHelper(5));

  const light = new THREE.PointLight(0xff00ff, .5);
  light.position.set(10, 10, 10);
  scene.add(light);

  const camera = new THREE.PerspectiveCamera(
    75,
    rect.width / rect.height,
    0.1,
    1000
  );

  camera.position.z = 6.5;
  camera.position.y = 2.5;
  camera.position.x = 2.5;

  /**
   * WebGLRenderer is the most used, fastest, and most supported renderer.
   */
  const renderer1 = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
  renderer1.setSize(rect.width, rect.height);

  const controls = new OrbitControls( camera, renderer1.domElement );

  const boxGeometry = new THREE.BoxGeometry();
  const torusKnotGeometry = new THREE.TorusKnotGeometry();
  const torusGeometry = new THREE.TorusGeometry();
  const planeGeometry = new THREE.PlaneGeometry();
  const sphereGeometry = new THREE.SphereGeometry();

  /**
   * is an extension of the MeshStandardMaterial which gives more
   * reflectivity options.
   */
  const material = new THREE.MeshPhysicalMaterial({
    transparent: true,
    opacity: 0.5,
  })

  const cube = new THREE.Mesh(boxGeometry, material)
  const torusKnot = new THREE.Mesh(torusKnotGeometry, material)
  const torus = new THREE.Mesh(torusGeometry, material)
  const plane = new THREE.Mesh(planeGeometry, material)
  const sphere = new THREE.Mesh(sphereGeometry, material)

  scene.add(cube)
  scene.add(torusKnot)
  scene.add(torus)
  scene.add(plane)
  scene.add(sphere)

  cube.position.x = -6;
  torusKnot.position.x = -3;
  torus.position.x = 0.75;
  plane.position.x = 3;
  sphere.position.x = 5;

  window.addEventListener('resize', onWindowResize, false)
  function onWindowResize() {
    camera.aspect = rect.width / rect.height;
    camera.updateProjectionMatrix();

    renderer1.setSize(rect.width, rect.height);
    
    render();
  }

  const data = {
    color: material.color.getHex(),
    emissive: material.emissive.getHex(),
  }

  // dynamic import needed because svelte errors since dat.gui uses window before
  // mount...however, this causes multiple renders of the dat.gui...just refresh
  // the page to get rid of the extra ones...
  const gui = new (await import('dat.gui')).GUI();

  const cubeFolder = gui.addFolder('Cube');
  cubeFolder.open();

  const cubeRotationFolder = cubeFolder.addFolder('Rotation');
  cubeRotationFolder.add(cube.rotation, 'x', 0, Math.PI * 2);
  cubeRotationFolder.add(cube.rotation, 'y', 0, Math.PI * 2);
  cubeRotationFolder.add(cube.rotation, 'z', 0, Math.PI * 2);

  const MeshFolder = gui.addFolder('Material');
  MeshFolder.open();
  MeshFolder.add(material, 'wireframe');
  MeshFolder.add(material, 'transparent');
  MeshFolder.add(material, 'opacity', 0, 1, 0.01);
  MeshFolder.add(material, 'depthTest');
  MeshFolder.add(material, 'depthWrite');
  MeshFolder.add(material, 'visible');
  MeshFolder.add(material, 'side', {
    FrontSide: THREE.FrontSide,
    BackSide: THREE.BackSide,
    DoubleSide: THREE.DoubleSide,
  }).onChange(() => {
    material.side = Number(material.side) as THREE.Side;
    material.needsUpdate = true
  });
  MeshFolder.add(material, 'flatShading').onChange(() => { material.needsUpdate = true });
  MeshFolder.addColor(data, 'emissive').onChange(() => {
    material.emissive.setHex(Number(data.emissive.toString().replace('#', '0x')));
    material.needsUpdate = true;
  });
  MeshFolder.add(material, 'roughness', 0, 1);
  MeshFolder.add(material, 'metalness', 0, 1);
  MeshFolder.add(material, 'reflectivity', 0, 1);
  MeshFolder.add(material, 'clearcoat', 0, 1, 0.01);
  MeshFolder.add(material, 'clearcoatRoughness', 0, 1, 0.01);

  const cameraFolder = gui.addFolder('Camera');

  const cameraPositionFolder = cameraFolder.addFolder('Position');
  cameraPositionFolder.add(camera.position, 'x', -10, 10);
  cameraPositionFolder.add(camera.position, 'y', -10, 10);
  cameraPositionFolder.add(camera.position, 'z', -10, 10);

  const cameraRotationFolder = cameraFolder.addFolder('Rotation');
  cameraRotationFolder.add(camera.rotation, 'x', 0, Math.PI * 2);
  cameraRotationFolder.add(camera.rotation, 'y', 0, Math.PI * 2);
  cameraRotationFolder.add(camera.rotation, 'z', 0, Math.PI * 2);

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
    renderer1.render(scene, camera)
  }

  animate()
})
</script>

<canvas id="c1"></canvas>
