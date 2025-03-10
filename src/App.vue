<script setup>
import * as THREE from "three";
import {onMounted, onUnmounted, setDevtoolsHook} from "vue";
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls.js";
import {DRACOLoader} from "three/examples/jsm/loaders/DRACOLoader.js";
import {GLTFLoader} from "three/examples/jsm/loaders/GLTFLoader.js";

// 初始化场景
const scene = new THREE.Scene();
// 初始化相机
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
)
camera.position.set(-3.23,2.98,4.06);
camera.aspect = window.innerWidth / window.innerHeight;
camera.updateProjectionMatrix();
// 初始化渲染器
const renderer = new THREE.WebGLRenderer({
  // 抗锯齿
  antialias: true,
})
renderer.setSize(window.innerWidth, window.innerHeight)
document.body.appendChild(renderer.domElement);

// 初始化控制器
const controls = new OrbitControls(camera, renderer.domElement);
// 阻尼
controls.enableDamping = true

// 初始化loader
const dracoLoader = new DRACOLoader()
dracoLoader.setDecoderPath("./draco/")
const gltfLoader = new GLTFLoader()
gltfLoader.setDRACOLoader(dracoLoader)

// 加载模型
gltfLoader.load("./model/scene.glb",(gltf)=>{
  const model = gltf.scene
  scene.add(model)
})

// 添加平行光源
const light = new THREE.DirectionalLight(0xffffff, 1)
light.position.set(0, 50, 0)
scene.add(light)

function render(){
  requestAnimationFrame(render);
  renderer.render(scene, camera);
  controls.update()
}
render()

// 添加清理
onUnmounted(() => {
  scene?.clear();
  renderer?.dispose();
});
</script>

<template>
  <!-- 移除scoped样式 -->
</template>

<style>
/* 改为全局样式 */
body { margin: 0; overflow: hidden; }
canvas {
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
</style>