<template>
  <div class="home-page">
    <div class="header">Three.js</div>
    <div ref="canvas" class="canvas"></div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

const canvas = ref(null);

onMounted(() => {
  // Create scene and camera
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    50,
    canvas.value.clientWidth / canvas.value.clientHeight,
    0.1,
    1000,
  );
  camera.position.z = 8;

  // Create renderer
  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(canvas.value.clientWidth, canvas.value.clientHeight);
  canvas.value.appendChild(renderer.domElement);

  // Add lighting
  const light = new THREE.HemisphereLight(0xffffff, 0x444444, 1);
  scene.add(light);

  // Enable mouse controls + auto-rotate
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.dampingFactor = 0.05;
  controls.autoRotate = true;
  controls.autoRotateSpeed = 16.0;
  controls.target.set(0, 0, 0);

  // Load .glb model
  const loader = new GLTFLoader();
  loader.load(
    new URL("../assets/planet.glb", import.meta.url).href,
    (gltf) => {
      gltf.scene.rotation.z = Math.PI / 5;
      gltf.scene.rotation.x = -Math.PI / 10;
      scene.add(gltf.scene);
    },
    undefined,
    (error) => {
      console.error("Failed to load model:", error);
    },
  );

  // Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    controls.update(); // required for OrbitControls
    renderer.render(scene, camera);
  };
  animate();
});
</script>

<style scoped>
.home-page {
  width: 100%;
  height: 100%;
  background: #24282f;
  display: flex;
  flex-direction: column;

  .header {
    height: 60px;
    background: #171d25;
    color: white;
    line-height: 60px;
    font-size: 24px;
    font-weight: 400;
    padding-left: 20px;
  }

  .canvas {
    flex: 1;
  }
}
</style>
