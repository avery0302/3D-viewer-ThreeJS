<template>
  <div class="home-page">
    <canvas
      ref="canvasRef"
      class="canvas"
      @dragover.prevent
      @drop.prevent="handleDrop"
    ></canvas>
    <div class="tips">Drag glb file to view it</div>
    <div class="header">3D viewer</div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

const canvasRef = ref(null);
let controls = null;
let camera = null;
let renderer = null;
let scene = null;
let loader = null;
let light = null;

function initScene() {
  // Create scene and camera
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    50,
    canvasRef.value.clientWidth / canvasRef.value.clientHeight,
    0.1,
    1000,
  );
  camera.position.set(0, 0, 8);

  // Create renderer
  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value,
    antialias: true,
    alpha: true,
  });
  renderer.setSize(canvasRef.value.clientWidth, canvasRef.value.clientHeight);

  // Add lighting
  light = new THREE.HemisphereLight(0xffffff, 0x444444, 1);
  scene.add(light);

  initControls();
  loadModel();

  // Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    controls.update(); // required for OrbitControls
    renderer.render(scene, camera);
  };
  animate();
}

function handleDrop(e) {
  const file = e.dataTransfer.files[0];
  if (!file || !file.name.endsWith(".glb")) {
    alert("Please drop a .glb file");
    return;
  }

  const reader = new FileReader();
  reader.onload = function (event) {
    const arrayBuffer = event.target.result;

    loader.parse(arrayBuffer, "", (gltf) => {
      scene.clear();
      scene.add(light);
      controls.autoRotate = false;
      camera.position.set(0, 0, 8);
      camera.lookAt(0, 0, 0);

      scene.add(gltf.scene);
    });
  };
  reader.readAsArrayBuffer(file);
}

function initControls() {
  // Enable mouse controls + auto-rotate
  controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.autoRotate = true;
  controls.autoRotateSpeed = 8.0;
  controls.target.set(0, 0, 0);
}

function loadModel() {
  // Load .glb model
  loader = new GLTFLoader();
  loader.load(
    new URL("../assets/planet.glb", import.meta.url).href,
    (gltf) => {
      gltf.scene.rotation.z = Math.PI / 1.8;
      scene.add(gltf.scene);
    },
    undefined,
    (error) => {
      console.error("Failed to load model:", error);
    },
  );
}

onMounted(() => {
  initScene();
});
</script>

<style lang="scss" scoped>
.home-page {
  width: 100%;
  height: 100%;
  background: #24282f;
  display: flex;
  flex-direction: column;
  font-weight: 400;

  .header {
    height: 80px;
    background: #171d25;
    color: white;
    line-height: 80px;
    font-size: 24px;
    padding-left: 20px;
  }

  .canvas {
    flex: 1;
  }

  .tips {
    width: 100%;
    height: 60px;
    color: white;
    text-align: center;
    font-size: 20px;
    line-height: 60px;
    font-weight: 300;
    position: absolute;
    bottom: 80px;
  }
}
</style>
