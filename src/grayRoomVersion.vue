<script setup>
import { onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
const textureLoader = new THREE.TextureLoader()

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

// 初始化场景
async function initScene() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0xffffff)

  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.5, 1000)
  camera.position.set(0, baseHeight, 0)

  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  document.body.appendChild(renderer.domElement)

  // 创建房间
  // 大理石墙面材质
  const wallTexture = await textureLoader.loadAsync(WALL_TEXTURE_PATH).catch(error => {
    console.error('墙面纹理加载失败:', error)
    return new THREE.Texture() // 返回空白纹理
  })
  wallTexture.wrapS = wallTexture.wrapT = THREE.RepeatWrapping
  wallTexture.repeat.set(4, 2)
  const roomGeometry = new THREE.BoxGeometry(roomSize, 5, roomSize)
  const roomMaterial = new THREE.MeshPhongMaterial({
    color: 0xffffff,
    side: THREE.BackSide,
    depthWrite: true // 启用深度写入
  })
  const room = new THREE.Mesh(roomGeometry, roomMaterial)
  room.position.y = 2.5
  scene.add(room)

  // 地板
  const floorGeometry = new THREE.PlaneGeometry(roomSize, roomSize)
  const floorMaterial = new THREE.MeshPhongMaterial({
    color: 0xcccccc,
    side: THREE.DoubleSide,
    depthWrite: false // 关闭地板深度写入
  })
  const floor = new THREE.Mesh(floorGeometry, floorMaterial)
  floor.rotation.x = Math.PI / 2
  floor.position.y = 0.01 // 轻微抬高地板
  scene.add(floor)

  // 照明系统
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.8)
  scene.add(ambientLight)
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
  requestAnimationFrame(animate)
  const deltaTime = clock.getDelta()

  handleMovement()
  updateHeadBob(deltaTime)
  renderer.render(scene, camera)
}

// 窗口调整
function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(() => {
  initScene()
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