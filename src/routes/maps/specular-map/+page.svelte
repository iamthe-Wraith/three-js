<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import grid from '$assets/grid.png';

onMount(async () => {
  const main = document.querySelector('main') as HTMLCanvasElement;
  const rect = main.getBoundingClientRect();

  const scene = new THREE.Scene()
  scene.background = new THREE.Color(0x111111);
  scene.add(new THREE.AxesHelper(5));

  const light = new THREE.PointLight(0xffffff, 2);
  light.position.set(10, 10, 10);
  scene.add(light);

  const camera = new THREE.PerspectiveCamera(
    75,
    rect.width / rect.height,
    0.1,
    1000
  );

  camera.position.z = 2;

  /**
   * WebGLRenderer is the most used, fastest, and most supported renderer.
   */
  const renderer1 = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
  renderer1.setSize(rect.width, rect.height);

  const controls = new OrbitControls( camera, renderer1.domElement );

  const planeGeometry = new THREE.PlaneGeometry();

  const material = new THREE.MeshPhongMaterial({
    transparent: true,
    opacity: 1,
  });

  const texture = new THREE.TextureLoader().load(grid);
  material.map = texture;

  const plane = new THREE.Mesh(planeGeometry, material)

  scene.add(plane)

  window.addEventListener('resize', onWindowResize, false)
  function onWindowResize() {
    camera.aspect = rect.width / rect.height;
    camera.updateProjectionMatrix();

    renderer1.setSize(rect.width, rect.height);
    
    render();
  }

  const options = {
    combine: {
      MultiplyOperation: THREE.MultiplyOperation,
      MixOperation: THREE.MixOperation,
      AddOperation: THREE.AddOperation
    }
  }

  const data = {
    color: material.color.getHex(),
    emissive: material.emissive.getHex(),
    specular: material.specular.getHex(),
  }

  // dynamic import needed because svelte errors since dat.gui uses window before
  // mount...however, this causes multiple renders of the dat.gui...just refresh
  // the page to get rid of the extra ones...
  const gui = new (await import('dat.gui')).GUI();

  const planeFolder = gui.addFolder('Plane');
  planeFolder.open();

  const MeshFolder = gui.addFolder('Material');
  MeshFolder.open();
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
  MeshFolder.addColor(data, 'color').onChange(() => material.color.setHex(Number(data.color)));
  MeshFolder.add(material, 'wireframe');
  MeshFolder.addColor(data, 'emissive').onChange(() => {
    material.emissive.setHex(Number(data.emissive.toString().replace('#', '0x')));
    material.needsUpdate = true;
  });
  MeshFolder.addColor(data, 'specular').onChange(() => {
    material.specular.setHex(Number(data.specular.toString().replace('#', '0x')));
    material.needsUpdate = true;
  });
  MeshFolder.add(material, 'shininess', 0, 1024);
  MeshFolder.add(material, 'reflectivity', 0, 1);
  MeshFolder.add(material, 'refractionRatio', 0, 1);
  MeshFolder.add(material, 'combine', options.combine)
    .onChange(() => {
      material.side = Number(material.side) as THREE.Side;
      material.combine = Number(material.combine) as THREE.Combine;
      material.needsUpdate = true;
    })

  function updateMaterial() {
    material.side = Number(material.side) as THREE.Side;
    material.needsUpdate = true
  }

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
