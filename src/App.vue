<script setup>
import {CSS2DRenderer} from "three/examples/jsm/renderers/CSS2DRenderer.js";
import * as THREE from "three";
import { onMounted, onUnmounted } from "vue";
import {TextureLoader} from "three";

// 添加动画循环引用
let animateId = null;
let scene, camera, renderer;

async function createWallMaterial() {
  const loader = new TextureLoader()
  try {
    const [map, normalMap, roughnessMap, displacementMap] = await Promise.all([
      loader.loadAsync('/Marble001_1K-JPG/Marble001_1K-JPG_Color.jpg'),
      loader.loadAsync('/Marble001_1K-JPG/Marble001_1K-JPG_NormalGL.jpg'),
      loader.loadAsync('/Marble001_1K-JPG/Marble001_1K-JPG_Roughness.jpg'),
      loader.loadAsync('/Marble001_1K-JPG/Marble001_1K-JPG_Displacement.jpg')
    ])

    return new THREE.MeshPhysicalMaterial({
      map,
      normalMap,
      roughnessMap,
      displacementMap,
      displacementScale: 0.01,
      normalScale: new THREE.Vector2(0.5, 0.5),
      roughness: 0.2,
      metalness: 0.0
      // 移除可能引起问题的clearcoat参数
    })
    return new THREE.MeshPhongMaterial({ color: 0xcccccc })
  } catch (error) {
    console.error('材质加载失败:', error)
    return new THREE.MeshPhongMaterial({ color: 0xcccccc })
  }
}

async function init() {
  // 初始化三维场景
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  renderer = new THREE.WebGLRenderer({ antialias: true });

  // 设置渲染器
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0xf0f0f0); // 添加背景色方便调试
  document.body.appendChild(renderer.domElement);

  // 调整相机位置和朝向
  camera.position.set(0, 0, 10);  // 修改相机初始位置
  camera.lookAt(0, 0, 0);        // 确保看向场景中心

  // 添加坐标系辅助（调试用）
  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  // 创建墙壁
  const wallGeometry = new THREE.PlaneGeometry(1200, window.innerHeight)
  const wallMaterial = new THREE.MeshPhongMaterial({
    color: 0x000000,
    side: THREE.DoubleSide
  })
  const wall = new THREE.Mesh(wallGeometry, wallMaterial)
  wall.rotation.y = Math.PI; // 正确旋转方向
  wall.position.z = -5;      // 调整到相机前方
  scene.add(wall);

  // 添加基础灯光
  // const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
  // scene.add(ambientLight);

  // 启动动画循环
  animate();
}

// 添加动画循环
function animate() {
  animateId = requestAnimationFrame(animate);
  renderer.render(scene, camera);

  // 添加简单旋转调试（可选）
  // scene.children[1].rotation.z += 0.01;
}

onMounted(() => {
  init()
})

// 添加清理
onUnmounted(() => {
  cancelAnimationFrame(animateId);
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
canvas { position: fixed; top: 0; left: 0; }
</style>