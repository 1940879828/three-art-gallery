<script setup>
import {onMounted, onUnmounted, ref} from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'
// 添加ref确保初始化顺序
const isInitialized = ref(false)

let renderer, scene, camera
const moveSpeed = 0.08
const sensitivity = 0.002
const keys = {}
const roomSize = 10
const wallThickness = 0.2

// 走路效果参数
let bobPhase = 0
const bobAmplitude = 0.04
const bobFrequency = 0.6
const baseHeight = 1.6
let isMoving = false

// 添加纹理加载器
const textureLoader = new THREE.TextureLoader()
const loader = new GLTFLoader()

// 初始化场景
async function initScene() {
  // 先创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  document.body.appendChild(renderer.domElement)

  // 再加载其他资源
  scene = new THREE.Scene()
  await createMuseumStructure()

  // 添加基础照明系统
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8)
  scene.add(ambientLight)

  // 最后创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.5, 1000)
  camera.position.set(0, 1.6, -5) // 向后移动避免穿墙
  camera.lookAt(0, 1.6, 0)
  camera.getWorldDirection(new THREE.Vector3())
}



// 修改纹理路径
const WALL_TEXTURE_PATH = '/Marble001_1K-JPG/Marble001_1K-JPG_Color.jpg'
const FLOOR_TEXTURE_PATH = '/Marble002_1K-JPG/Marble002_1K-JPG_Color.jpg'

// 创建博物馆建筑结构
async function createMuseumStructure() {
  // 大理石墙面材质
  const wallTexture = await textureLoader.loadAsync(WALL_TEXTURE_PATH).catch(error => {
    console.error('墙面纹理加载失败:', error)
    return new THREE.Texture() // 返回空白纹理
  })
  wallTexture.wrapS = wallTexture.wrapT = THREE.RepeatWrapping
  wallTexture.repeat.set(4, 2)

  // 博物馆主厅
  const hallGeometry = new THREE.BoxGeometry(roomSize, 8, roomSize*1.5)
  const hallMaterial = new THREE.MeshPhongMaterial({
    map: wallTexture,
    side: THREE.BackSide,
    bumpScale: 0.02
  })
  const mainHall = new THREE.Mesh(hallGeometry, hallMaterial)
  mainHall.position.y = 4
  scene.add(mainHall)

  // 大理石地板
  const floorTexture = await textureLoader.loadAsync(FLOOR_TEXTURE_PATH).catch(error => {
    console.error('墙面纹理加载失败2:', error)
    return new THREE.Texture() // 返回空白纹理
  })
  floorTexture.wrapS = floorTexture.wrapT = THREE.RepeatWrapping
  floorTexture.repeat.set(8, 8)

  const floor = new THREE.Mesh(
    new THREE.PlaneGeometry(roomSize*2, roomSize*2),
    new THREE.MeshPhongMaterial({
      map: floorTexture,
      side: THREE.DoubleSide
    })
  )
  floor.rotation.x = Math.PI/2
  floor.position.y = 0.1
  scene.add(floor)

  // 添加展厅立柱
  createPillars()
}

// 创建古典立柱
function createPillars() {
  const pillarGeometry = new THREE.CylinderGeometry(0.8, 1, 6, 16)
  const pillarMaterial = new THREE.MeshPhongMaterial({
    color: 0xcdba96,
    specular: 0x222222
  })

  // 排列立柱
  const positions = [-6, -3, 0, 3, 6]
  positions.forEach(x => {
    positions.forEach(z => {
      if(Math.abs(x) === Math.abs(z)) return
      const pillar = new THREE.Mesh(pillarGeometry, pillarMaterial)
      pillar.position.set(x, 3, z)
      scene.add(pillar)
    })
  })
}

// 鼠标移动处理
function handleMouseMove(e) {
  if (document.pointerLockElement === renderer.domElement) {
    const { movementX } = e

    // 水平旋转（左右转头）
    camera.rotation.y -= movementX * sensitivity
  }
}

// 锁定鼠标指针
function requestPointerLock() {
  renderer.domElement.requestPointerLock()
}

// 键盘事件处理
function handleKeyDown(e) {
  keys[e.key.toLowerCase()] = true
  isMoving = true
}

function handleKeyUp(e) {
  keys[e.key.toLowerCase()] = false
  isMoving = Object.values(keys).some(v => v)
}

// 移动和碰撞检测
function handleMovement() {
  if (!camera) return // 添加安全判断

  const forward = new THREE.Vector3()
  const right = new THREE.Vector3()
  camera.getWorldDirection(forward)
  forward.y = 0
  forward.normalize()
  right.crossVectors(forward, camera.up).normalize()

  const moveDelta = new THREE.Vector3()
  if (keys.w) moveDelta.add(forward.multiplyScalar(moveSpeed))
  if (keys.s) moveDelta.add(forward.multiplyScalar(-moveSpeed))
  if (keys.a) moveDelta.add(right.multiplyScalar(-moveSpeed))
  if (keys.d) moveDelta.add(right.multiplyScalar(moveSpeed))

  // 碰撞检测
  const newPos = camera.position.clone().add(moveDelta)
  const halfSize = roomSize/2 - wallThickness
  newPos.x = THREE.MathUtils.clamp(newPos.x, -halfSize, halfSize)
  newPos.z = THREE.MathUtils.clamp(newPos.z, -halfSize, halfSize)

  camera.position.copy(newPos)
}

// 走路颠簸效果
function updateHeadBob(deltaTime) {
  if (isMoving) {
    bobPhase += deltaTime * bobFrequency
    const verticalOffset = Math.sin(bobPhase) * bobAmplitude
    camera.position.y = baseHeight + verticalOffset
  } else {
    // 平滑复位
    camera.position.y += (baseHeight - camera.position.y) * 0.1
    bobPhase = 0
  }
}

// 动画循环
let clock = new THREE.Clock()
function animate() {
  if (!isInitialized.value) {
    requestAnimationFrame(animate)
    return
  }

  const deltaTime = clock.getDelta()
  handleMovement()
  updateHeadBob(deltaTime)
  renderer.render(scene, camera)
  requestAnimationFrame(animate)
}


// 窗口调整
function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(async() => {
  await initScene()
  animate()
  window.addEventListener('resize', onWindowResize)
  window.addEventListener('keydown', handleKeyDown)
  window.addEventListener('keyup', handleKeyUp)
  renderer.domElement.addEventListener('click', requestPointerLock)
  document.addEventListener('mousemove', handleMouseMove)
})

onUnmounted(() => {
  window.removeEventListener('resize', onWindowResize)
  window.removeEventListener('keydown', handleKeyDown)
  window.removeEventListener('keyup', handleKeyUp)
  document.removeEventListener('mousemove', handleMouseMove)
  document.body.removeChild(renderer.domElement)
})
</script>

<template>
  <!-- 空模板 -->
</template>

<style scoped>
body {
  margin: 0;
  overflow: hidden;
}
canvas {
  cursor: crosshair;
}
</style>