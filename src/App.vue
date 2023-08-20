<script setup>
import * as THREE from 'three';
import {onMounted, ref} from "vue";
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const webGl = ref();
let controls = null;
let scene = null;
let camera = null;
let renderer = null;
const aspect = window.innerWidth / window.innerHeight;

const setCanvas = () => {
  //scene
  scene = new THREE.Scene();
  scene.background = new THREE.TextureLoader().load("/8k_stars.jpg");

  //sphere
  const geometry = new THREE.SphereGeometry( 5, 60, 60 );
  const material = new THREE.MeshStandardMaterial( {
   map: new THREE.TextureLoader().load('/8k_mars.jpg')
  });
  const sphere = new THREE.Mesh( geometry, material );
  scene.add(sphere);

  //light
  const light = new THREE.PointLight(0xffffff, 5000);
  light.position.set(20, 20, 20);
  scene.add(light);

  //camera
  camera = new THREE.PerspectiveCamera(45, aspect, 0.1, 100);
  scene.add(camera);
  camera.add(light);
  camera.position.z = 19;

  //renderer
  const canvas = webGl.value;
  renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.render(scene, camera);

  controls = new OrbitControls( camera, renderer.domElement );
  controls.maxDistance = 40;
  controls.minDistance = 8;
  controls.minPolarAngle = 0;
  controls.maxPolarAngle = Math.PI;
  controls.mouseButtons.RIGHT = null;
  controls.zoomSpeed = 0.8;
  controls.autoRotate = true;
  controls.enableDamping = true;
  controls.rotateSpeed = 0.5;
  controls.update();
}

function animate() {
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

onMounted(() => {
  setCanvas();
  animate();
})
</script>

<template>
  <canvas ref="webGl" class="canvas" />
</template>

<style>

</style>
