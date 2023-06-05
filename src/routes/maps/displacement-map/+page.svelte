<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import worldColorFile from '$assets/specularMaps/worldColour.5400x2700.jpg';
  import displacementMapFile from '$assets/displacementMaps/gebco_bathy.5400x2700_8bit.jpg';

  onMount(async () => {
    const main = document.querySelector('main') as HTMLCanvasElement;
    const rect = main.getBoundingClientRect();

    const scene = new THREE.Scene()
    scene.add(new THREE.AxesHelper(5))

    const light = new THREE.PointLight(0xffffff, 2)
    light.position.set(0, 2, 5)
    scene.add(light)

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    );

    camera.position.z = 1;

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height);

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.screenSpacePanning = true //so that panning up and down doesn't zoom in/out
    //controls.addEventListener('change', render)

    const planeGeometry = new THREE.PlaneGeometry(3.6, 1.8, 360, 180);

    const material = new THREE.MeshPhongMaterial()

    //const texture = new THREE.TextureLoader().load("img/grid.png")
    const texture = new THREE.TextureLoader().load(worldColorFile);
    material.map = texture
    // const envTexture = new THREE.CubeTextureLoader().load(["img/px_50.png", "img/nx_50.png", "img/py_50.png", "img/ny_50.png", "img/pz_50.png", "img/nz_50.png"])
    // const envTexture = new THREE.CubeTextureLoader().load(["img/px_eso0932a.jpg", "img/nx_eso0932a.jpg", "img/py_eso0932a.jpg", "img/ny_eso0932a.jpg", "img/pz_eso0932a.jpg", "img/nz_eso0932a.jpg"])
    // envTexture.mapping = THREE.CubeReflectionMapping
    // material.envMap = envTexture

    //const specularTexture = new THREE.TextureLoader().load("img/earthSpecular.jpg")
    // material.specularMap = specularTexture

    /**
     * in displacement maps, darker colors indicate lower elevation
     * and lighter colors indicate higher elevation
     */
    const displacementMap = new THREE.TextureLoader().load(displacementMapFile);
    material.displacementMap = displacementMap

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
    materialFolder.add(material, 'transparent').onChange(() => material.needsUpdate = true)
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
        specular: material.specular.getHex(),
    }

    const meshPhongMaterialFolder = gui.addFolder('THREE.meshPhongMaterialFolder')

    meshPhongMaterialFolder.addColor(data, 'color').onChange(() => {
        material.color.setHex(Number(data.color.toString().replace('#', '0x')))
    })
    meshPhongMaterialFolder.addColor(data, 'emissive').onChange(() => {
        material.emissive.setHex(
            Number(data.emissive.toString().replace('#', '0x'))
        )
    })
    meshPhongMaterialFolder.addColor(data, 'specular').onChange(() => {
        material.specular.setHex(
            Number(data.specular.toString().replace('#', '0x'))
        )
    })
    meshPhongMaterialFolder.add(material, 'shininess', 0, 1024)
    meshPhongMaterialFolder.add(material, 'wireframe', true);
    meshPhongMaterialFolder
        .add(material, 'flatShading')
        .onChange(() => updateMaterial())
    meshPhongMaterialFolder.add(material, 'reflectivity', 0, 1)
    meshPhongMaterialFolder.add(material, 'refractionRatio', 0, 1)
    /**
     * using displacementScale will increase or reduce the range of the displacement map
     * ie. lower value will reduce the range of the displacement map values so the
     * elevation changes will not be as drastic.
    */
    meshPhongMaterialFolder.add(material, 'displacementScale', 0, 1, 0.01)
    meshPhongMaterialFolder.add(material, 'displacementBias', -1, 1, 0.01)
    meshPhongMaterialFolder.open()

    function updateMaterial() {
        material.side = Number(material.side) as THREE.Side
        material.needsUpdate = true
    }

    const planeData = {
        width: 3.6,
        height: 1.8,
        widthSegments: 360,
        heightSegments: 180
    };
    const planePropertiesFolder = gui.addFolder("PlaneGeometry")
    //planePropertiesFolder.add(planeData, 'width', 1, 30).onChange(regeneratePlaneGeometry)
    //planePropertiesFolder.add(planeData, 'height', 1, 30).onChange(regeneratePlaneGeometry)

    /**
     * adjust these 2 values to add additional vertices to the geometry.
     */
    planePropertiesFolder.add(planeData, 'widthSegments', 1, 360).onChange(regeneratePlaneGeometry)
    planePropertiesFolder.add(planeData, 'heightSegments', 1, 180).onChange(regeneratePlaneGeometry)
    planePropertiesFolder.open()

    const textureFolder = gui.addFolder("Texture")
    /**
     * repeat is the number of times the texture is repeated in the x and y directions.
     * ie. will reduce the amount of the texture that is visible
    */
    textureFolder.add(texture.repeat, 'x', 0.1, 1, 0.1)
    textureFolder.add(texture.repeat, 'y', 0.1, 1, 0.1)
    /**
     * offset is the amount of the texture that is offset in the x and y directions.
     * think of this like scrolling the texture, where this value is the position
     * of the scroll.`
    */
    textureFolder.add(texture.center, 'x', 0, 1, 0.001)
    textureFolder.add(texture.center, 'y', 0, 1, 0.001)
    textureFolder.open();

    function regeneratePlaneGeometry() {
        let newGeometry = new THREE.PlaneGeometry(
            planeData.width,
            planeData.height, 
            planeData.widthSegments, 
            planeData.heightSegments
        )
        plane.geometry.dispose()
        plane.geometry = newGeometry
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
