<script setup>
import {CSS2DObject, CSS2DRenderer} from "three/examples/jsm/renderers/CSS2DRenderer.js";
import * as THREE from "three";
import {onMounted} from "vue";
import {TextureLoader} from "three";

let currentIndex = 0;
const paintings = [];
const paintingWidth = 3;
const spacing = 1;
const offsetX = 0;
let container = new THREE.Group();

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
  } catch (error) {
    console.error('材质加载失败:', error)
    return new THREE.MeshPhongMaterial({ color: 0xcccccc })
  }
}

async function init() {
  // 场景设置
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  const labelRenderer = new CSS2DRenderer();

  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  labelRenderer.setSize(window.innerWidth, window.innerHeight);
  labelRenderer.domElement.style.position = 'absolute';
  labelRenderer.domElement.style.top = '0';
  document.body.appendChild(labelRenderer.domElement);

  // 灯光
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
  directionalLight.position.set(0, 3, 5);
  scene.add(directionalLight);

  // 创建墙壁
  const wallMaterial = await createWallMaterial()
  const wall = new THREE.Mesh(
    new THREE.PlaneGeometry(20, 10),
    wallMaterial
  );
  wall.rotation.y = Math.PI; // 让墙壁面向相机
  wall.position.z = -5;
  scene.add(wall);

  // 相机位置
  camera.position.set(0, 1.5, 3);

  // 创建画作
  // createPaintings(scene).then(() => {
  //   scene.add(container);
  //   arrangePaintings();
  // });

  // 事件监听
  // document.getElementById('prevBtn').addEventListener('click', prevPainting);
  // document.getElementById('nextBtn').addEventListener('click', nextPainting);
  // window.addEventListener('resize', onWindowResize);

  // function onWindowResize() {
  //   camera.aspect = window.innerWidth / window.innerHeight;
  //   camera.updateProjectionMatrix();
  //   renderer.setSize(window.innerWidth, window.innerHeight);
  //   labelRenderer.setSize(window.innerWidth, window.innerHeight);
  // }

  // 动画循环
  // function animate() {
  //   requestAnimationFrame(animate);
  //   renderer.render(scene, camera);
  //   labelRenderer.render(scene, camera);
  // }
  // animate();
}

async function createPaintings() {
  const textureLoader = new THREE.TextureLoader();
  // 示例图片（替换为你的图片路径）
  const paintingImages = [
    { url: 'https://example.com/painting1.jpg', title: '作品1' },
    { url: 'https://example.com/painting2.jpg', title: '作品2' },
    { url: 'https://example.com/painting3.jpg', title: '作品3' },
    { url: 'https://example.com/painting4.jpg', title: '作品4' }
  ];

  for (let i = 0; i < paintingImages.length; i++) {
    const texture = await textureLoader.loadAsync(paintingImages[i].url);
    texture.colorSpace = THREE.SRGBColorSpace;

    // 画框
    const frame = new THREE.Mesh(
      new THREE.BoxGeometry(paintingWidth, paintingWidth * 1.2, 0.1),
      new THREE.MeshPhongMaterial({
        color: 0x8B4513, // 木质颜色
        map: texture
      })
    );
    frame.position.y = 0.6; // 调整画作高度

    // 文字标签
    const labelDiv = document.createElement('div');
    labelDiv.textContent = paintingImages[i].title;
    labelDiv.style.color = 'black';
    labelDiv.style.fontSize = '20px';
    labelDiv.style.textAlign = 'center';
    const label = new CSS2DObject(labelDiv);
    label.position.set(0, -paintingWidth * 0.7, 0); // 文字位置

    const paintingGroup = new THREE.Group();
    paintingGroup.add(frame, label);
    container.add(paintingGroup);
    paintings.push(paintingGroup);
  }
}

function arrangePaintings() {
  const totalWidth = (paintingWidth + spacing) * paintings.length;
  const startX = -totalWidth / 2 + paintingWidth / 2;

  paintings.forEach((painting, index) => {
    painting.position.x = startX + index * (paintingWidth + spacing) + offsetX;
  });
}

function nextPainting() {
  if (currentIndex < paintings.length - 1) {
    currentIndex++;
    animateTransition();
  }
}

function prevPainting() {
  if (currentIndex > 0) {
    currentIndex--;
    animateTransition();
  }
}

function animateTransition() {
  const targetX = -currentIndex * (paintingWidth + spacing);

  // 使用简单动画
  const duration = 500; // 动画时长（毫秒）
  const startTime = Date.now();
  const startX = container.position.x;

  function update() {
    const elapsed = Date.now() - startTime;
    const progress = Math.min(elapsed / duration, 1);

    container.position.x = startX + (targetX - startX) * progress;

    if (progress < 1) {
      requestAnimationFrame(update);
    }
  }

  requestAnimationFrame(update);
}

onMounted(()=>{
  setTimeout(()=>{
    init();
  },50)
})
</script>

<template>
  <div id="prevBtn" class="button">←</div>
  <div id="nextBtn" class="button">→</div>
</template>

<style scoped>
body {
  margin: 0;
  overflow: hidden;
}
canvas { position: fixed; top: 0; left: 0 }
.button {
  position: fixed;
  top: 50%;
  transform: translateY(-50%);
  font-size: 40px;
  cursor: pointer;
  user-select: none;
  background: rgba(255,255,255,0.5);
  padding: 10px 20px;
  border-radius: 5px;
}
#prevBtn { left: 20px; }
#nextBtn { right: 20px; }
</style>