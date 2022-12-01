<script setup lang="ts">
import {
  Scene,
  PerspectiveCamera,
  Mesh,
  SphereGeometry,
  MeshBasicMaterial,
  WebGLRenderer,
  Color,
  Fog,
  AmbientLight,
} from 'three'
import { Ref } from 'vue'
import { useWindowSize } from '@vueuse/core'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

let renderer: WebGLRenderer
let controls: OrbitControls

const experience: Ref<HTMLCanvasElement | null> = ref(null)

const { width, height } = useWindowSize()
const aspectRatio = computed(() => width.value / height.value)

const bgColor = new Color('#E1F0C2')

const scene = new Scene()

scene.fog = new Fog(bgColor, 0.1, 75)
scene.background = bgColor

const camera = new PerspectiveCamera(75, aspectRatio.value, 0.1, 1000)
camera.position.set(0, 2, 4)

scene.add(camera)

const ambientLight = new AmbientLight(0xffffff, 1)
scene.add(ambientLight)

const { load } = useGLTFModel()

const { scene: model } = await load('/nuxty/nuxty.gltf')

scene.add(model)

function updateCamera() {
  camera.aspect = aspectRatio.value
  camera.updateProjectionMatrix()
}

function updateRenderer() {
  renderer.setSize(width.value, height.value)
  renderer.render(scene, camera)
}

function setRenderer() {
  if (experience.value) {
    renderer = new WebGLRenderer({ canvas: experience.value, alpha: true })
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
    controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    updateRenderer()
  }
}

watch(aspectRatio, () => {
  updateCamera()
  updateRenderer()
})

onMounted(() => {
  setRenderer()
  loop()
})

const loop = () => {
  controls.update()
  renderer.render(scene, camera)
  requestAnimationFrame(loop)
}
</script>
<template>
  <div>
    <canvas ref="experience" />
  </div>
</template>
