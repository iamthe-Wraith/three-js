<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import gridFile from '$assets/grid.png';
  import specularGrayscaleTextureFile from '$assets/specularMaps/grayscale-test.png';
  import specularEarthTextureFile from '$assets/specularMaps/earthSpecular.jpg';
  import worldColorFile from '$assets/specularMaps/worldColour.5400x2700.jpg';
  import envPX from '$assets/specularMaps/px_eso0932a.jpg';
  import envNX from '$assets/specularMaps/nx_eso0932a.jpg';
  import envPY from '$assets/specularMaps/py_eso0932a.jpg';
  import envNY from '$assets/specularMaps/ny_eso0932a.jpg';
  import envPZ from '$assets/specularMaps/pz_eso0932a.jpg';
  import envNZ from '$assets/specularMaps/nz_eso0932a.jpg';

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    const light = new THREE.PointLight(0xffffff, 2)
    light.position.set(0, 5, 10)
    scene.add(light)

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    );

    camera.position.z = 3;

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.screenSpacePanning = true //so that panning up and down doesn't zoom in/out
    //controls.addEventListener('change', render)

    const planeGeometry = new THREE.PlaneGeometry(3.6, 1.8)

    const material = new THREE.MeshPhysicalMaterial({})

    //const texture = new THREE.TextureLoader().load("img/grid.png")
    const texture = new THREE.TextureLoader().load(worldColorFile)
    material.map = texture
    // const envTexture = new THREE.CubeTextureLoader().load(["img/px_50.png", "img/nx_50.png", "img/py_50.png", "img/ny_50.png", "img/pz_50.png", "img/nz_50.png"])
    const envTexture = new THREE.CubeTextureLoader().load([
      envPX,
      envNX,
      envPY,
      envNY,
      envPZ,
      envNZ,
    ])
    envTexture.mapping = THREE.CubeReflectionMapping
    material.envMap = envTexture

    //const specularTexture = new THREE.TextureLoader().load("img/grayscale-test.png")
    const specularTexture = new THREE.TextureLoader().load(specularEarthTextureFile)
    material.roughnessMap = specularTexture
    material.metalnessMap = specularTexture

    const plane: THREE.Mesh = new THREE.Mesh(planeGeometry, material)
    scene.add(plane)

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight
      camera.updateProjectionMatrix()
      renderer.setSize(window.innerWidth, window.innerHeight)
      render()
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    const options = {
      side: {
        FrontSide: THREE.FrontSide,
        BackSide: THREE.BackSide,
        DoubleSide: THREE.DoubleSide,
      },
    }
    const gui = new (await import('dat.gui')).GUI();

    const materialFolder = gui.addFolder('THREE.Material')
    materialFolder.add(material, 'transparent')
    materialFolder.add(material, 'opacity', 0, 1, 0.01)
    materialFolder.add(material, 'depthTest')
    materialFolder.add(material, 'depthWrite')
    materialFolder
      .add(material, 'alphaTest', 0, 1, 0.01)
      .onChange(() => updateMaterial())
    materialFolder.add(material, 'visible')
    materialFolder
      .add(material, 'side', options.side)
      .onChange(() => updateMaterial())
    //materialFolder.open()

    const data = {
      color: material.color.getHex(),
      emissive: material.emissive.getHex(),
    }

    const meshPhysicalMaterialFolder = gui.addFolder(
      'THREE.meshPhysicalMaterialFolder'
    )

    meshPhysicalMaterialFolder.addColor(data, 'color').onChange(() => {
      material.color.setHex(Number(data.color.toString().replace('#', '0x')))
    })
    meshPhysicalMaterialFolder.addColor(data, 'emissive').onChange(() => {
    material.emissive.setHex(
        Number(data.emissive.toString().replace('#', '0x'))
    )
    })
    meshPhysicalMaterialFolder.add(material, 'wireframe')
    meshPhysicalMaterialFolder
      .add(material, 'flatShading')
      .onChange(() => updateMaterial())
    meshPhysicalMaterialFolder.add(material, 'reflectivity', 0, 1)
    meshPhysicalMaterialFolder.add(material, 'envMapIntensity', 0, 1)
    meshPhysicalMaterialFolder.add(material, 'roughness', 0, 1)
    meshPhysicalMaterialFolder.add(material, 'metalness', 0, 1)
    meshPhysicalMaterialFolder.add(material, 'clearcoat', 0, 1, 0.01)
    meshPhysicalMaterialFolder.add(material, 'clearcoatRoughness', 0, 1, 0.01)
    meshPhysicalMaterialFolder.open()

    function updateMaterial() {
      material.side = Number(material.side) as THREE.Side
      material.needsUpdate = true
    }

    function animate() {
      requestAnimationFrame(animate)

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
