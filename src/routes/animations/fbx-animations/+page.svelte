<script lang="ts" type="module">
  import { onMount } from 'svelte';
  import * as THREE from 'three'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
  import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader';
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

    const light = new THREE.PointLight()
    light.position.set(2.5, 7.5, 15)
    scene.add(light)

    const camera = new THREE.PerspectiveCamera(
      75,
      rect.width / rect.height,
      0.1,
      1000
    )
    camera.position.set(0.8, 1.4, 1.0)

    const renderer = new THREE.WebGLRenderer({ canvas: document.querySelector('#c1') as HTMLCanvasElement })
    renderer.setSize(rect.width, rect.height)

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.target.set(0, 1, 0)

    let mixer: THREE.AnimationMixer
    let modelReady = false
    const animationActions: THREE.AnimationAction[] = []
    let activeAction: THREE.AnimationAction
    let lastAction: THREE.AnimationAction
    const fbxLoader: FBXLoader = new FBXLoader()

    fbxLoader.load(
      `${assets}/vanguard-by-t-choonyung.fbx`,
      (object) => {
        object.scale.set(0.01, 0.01, 0.01)
        mixer = new THREE.AnimationMixer(object)

        const animationAction = mixer.clipAction(
            (object as THREE.Object3D).animations[0]
        )
        animationActions.push(animationAction)
        animationsFolder.add(animations, 'default')
        activeAction = animationActions[0]

        scene.add(object)

        // add an animation from another file
        fbxLoader.load(`${assets}/vanguard@bellydance.fbx`,
            (object) => {
              console.log("loaded belly dance")

              const animationAction = mixer.clipAction((object as THREE.Object3D).animations[0]);
              animationActions.push(animationAction)
              animationsFolder.add(animations, "bellydance")
              modelReady = true;
            },
            (xhr) => {
              console.log((xhr.loaded / xhr.total * 100) + '% loaded')
            },
            (error) => {
              console.log(error)
            }
        )
      },
      (xhr) => {
        console.log((xhr.loaded / xhr.total) * 100 + '% loaded')
      },
      (error) => {
        console.log(error)
      }
    )

    window.addEventListener('resize', onWindowResize, false)
    function onWindowResize() {
      camera.aspect = rect.width / rect.height
      camera.updateProjectionMatrix()
      renderer.setSize(rect.width, rect.height)
      render()
    }

    const stats = new Stats()
    document.body.appendChild(stats.dom)

    const animations = {
      default: function () {
        setAction(animationActions[0])
      },
      bellydance: function () {
        setAction(animationActions[1])
      },
    }

    const setAction = (toAction: THREE.AnimationAction) => {
      if (toAction != activeAction) {
        lastAction = activeAction
        activeAction = toAction
        // lastAction.stop() // will stop the animation abruptly
        lastAction.fadeOut(1) // will add a transition between the animations along with fadeIn below
        activeAction.reset()
        activeAction.fadeIn(1) // will add a transition between the animations along with fadeOut above
        activeAction.play()
      }
    }

    const gui = new (await import('dat.gui')).GUI();

    const animationsFolder = gui.addFolder('Animations')
    animationsFolder.open()

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

<canvas id="c1"></canvas>
