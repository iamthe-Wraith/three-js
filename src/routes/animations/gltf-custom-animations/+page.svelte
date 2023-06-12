<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { GLTFLoader, type GLTF } from 'three/examples/jsm/loaders/GLTFLoader';
  import Stats from 'three/examples/jsm/libs/stats.module';
  import { assets } from '$app/paths';

  /**
   * The FBX format is used to provide interoperability between digital content creation applications and game 
   * engines such as Blender, Maya, Autodesk, Unity, Unreal and many others. It supports many features such as 
   * 3D models, scene hierarchy, materials, lighting, animations, bones and more.
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
    camera.position.set(4, 4, 4)

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height)

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true

    let mixer: THREE.AnimationMixer
    let modelReady = false

    const gltfLoader = new GLTFLoader()

    const dropzone = document.getElementById('dropzone') as HTMLDivElement

    dropzone.ondragover = dropzone.ondragenter = function (evt) {
        evt.preventDefault()
    }
    dropzone.ondrop = function (evt: DragEvent) {
        evt.stopPropagation()
        evt.preventDefault()

        //clear the scene
        for (let i = scene.children.length - 1; i >= 0; i--) {
            scene.remove(scene.children[i])
        }
        //clear the checkboxes
        const myNode = document.getElementById('animationsPanel') as HTMLDivElement
        while (myNode.firstChild) {
            myNode.removeChild(myNode.lastChild as any)
        }

        const axesHelper = new THREE.AxesHelper(5)
        scene.add(axesHelper)

        const light1 = new THREE.DirectionalLight(new THREE.Color(0xffcccc))
        light1.position.set(-1, 1, 1)
        scene.add(light1)

        const light2 = new THREE.DirectionalLight(new THREE.Color(0xccffcc))
        light2.position.set(1, 1, 1)
        scene.add(light2)

        const light3 = new THREE.DirectionalLight(new THREE.Color(0xccccff))
        light3.position.set(0, -1, 0)
        scene.add(light3)

        const files = (evt.dataTransfer as DataTransfer).files
        const reader = new FileReader()
        reader.onload = function () {
            gltfLoader.parse(
                reader.result as string,
                '/',
                (gltf: GLTF) => {
                    console.log(gltf.scene)

                    mixer = new THREE.AnimationMixer(gltf.scene)

                    console.log(gltf.animations)

                    if (gltf.animations.length > 0) {
                        const animationsPanel = document.getElementById(
                            'animationsPanel'
                        ) as HTMLDivElement
                        const ul = document.createElement('UL') as HTMLUListElement
                        const ulElem = animationsPanel.appendChild(ul)

                        gltf.animations.forEach((a: THREE.AnimationClip, i) => {
                            const li = document.createElement('UL') as HTMLLIElement
                            const liElem = ulElem.appendChild(li)

                            const checkBox = document.createElement(
                                'INPUT'
                            ) as HTMLInputElement
                            checkBox.id = 'checkbox_' + i
                            checkBox.type = 'checkbox'
                            checkBox.addEventListener('change', (e: Event) => {
                                if ((e.target as HTMLInputElement).checked) {
                                    mixer
                                        .clipAction((gltf as any).animations[i])
                                        .play()
                                } else {
                                    mixer
                                        .clipAction((gltf as any).animations[i])
                                        .stop()
                                }
                            })
                            liElem.appendChild(checkBox)

                            const label = document.createElement(
                                'LABEL'
                            ) as HTMLLabelElement
                            label.htmlFor = 'checkbox_' + i
                            label.innerHTML = a.name
                            liElem.appendChild(label)
                        })

                        if (gltf.animations.length > 1) {
                            const btnPlayAll = document.getElementById(
                                'btnPlayAll'
                            ) as HTMLButtonElement
                            btnPlayAll.addEventListener('click', (e: Event) => {
                                mixer.stopAllAction()
                                gltf.animations.forEach(
                                    (a: THREE.AnimationClip) => {
                                        mixer.clipAction(a).play()
                                    }
                                )
                            })

                            btnPlayAll.style.display = 'block'
                        }
                    } else {
                        const animationsPanel = document.getElementById(
                            'animationsPanel'
                        ) as HTMLDivElement
                        animationsPanel.innerHTML = 'No animations found in model'
                    }

                    scene.add(gltf.scene)

                    const bbox = new THREE.Box3().setFromObject(gltf.scene)
                    controls.target.x = (bbox.min.x + bbox.max.x) / 2
                    controls.target.y = (bbox.min.y + bbox.max.y) / 2
                    controls.target.z = (bbox.min.z + bbox.max.z) / 2

                    modelReady = true
                },
                (error) => {
                    console.log(error)
                }
            )
        }
        reader.readAsArrayBuffer(files[0])
    }

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
        camera.aspect = rect.width / rect.height
        camera.updateProjectionMatrix()
        renderer.setSize(rect.width, rect.height)
        render()
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    const clock = new THREE.Clock()

    function animate() {
        requestAnimationFrame(animate)

        controls.update()

        if (modelReady) mixer.update(clock.getDelta())

        render()

        stats.update()
    }

    function render() {
        renderer.render(scene, camera)
    }

    animate()
  })
</script>

<div id="dropzone">
  <p>Drag (uncompressed) animated gLTF/gLB here</p>
</div>
<button id="btnPlayAll">Play All</button>
<div id="animationsPanel"></div>

<canvas id="c1"></canvas>

<style>
  body {
    overflow: hidden;
    margin: 0px;
  }

  #animationsPanel {
    position: absolute;
    top: 40px;
    right: 10px;
    width: 300px;
    height: auto;
    border: 1px solid white;
    color: white;
    font-family: monospace;
    font-size: 17px;
  }

  #animationsPanel ul {
    padding: 4px;
    list-style-type: none;
  }

  #btnPlayAll {
    position: absolute;
    top: 10px;
    right: 10px;
    width: 300px;
    display: none;
  }

  #dropzone {
    position: absolute;
    width: 140px;
    height: 90px;
    left: 10px;
    top: 50px;
    border: 10px dotted gray;
    background-color: lightgray;
    padding: 20px;
    font-family: monospace;
    font-size: 17px;
  }

  #dropzone p {
    pointer-events: none;
  }
</style>
