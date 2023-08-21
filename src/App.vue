<script setup>
import * as THREE from 'three';
import {onMounted, ref, watch, computed} from "vue";
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import {useWindowResize} from "@/hooks/useWindowResize";

const webGl = ref();
let controls = null;
let scene = null;
let camera = null;
let renderer = null;
let composer = null;
const {width, height} = useWindowResize();
const aspect = computed(() => {
  return width.value / height.value;
});
const params = {
  threshold: 0,
  strength: 3,
  radius: 1,
  exposure: 0.6
};

const setCanvas = () => {
  //scene
  scene = new THREE.Scene();

  //sphere
  const geometry = new THREE.SphereGeometry( 5, 60, 60 );
  const material = new THREE.MeshStandardMaterial( {
   map: new THREE.TextureLoader().load('/8k_mercury.jpg'),
  });
  const sphere = new THREE.Mesh(geometry, material);
  scene.add(sphere);

  //ring
  // const ringGeometry = new THREE.RingGeometry( 9, 6, 60 );
  // const ringMaterial = new THREE.MeshLambertMaterial({
  //   map: new THREE.TextureLoader().load('/8k_saturn_ring_alpha.png'),
  //   side: THREE.FrontSide
  // })
  // const ring = new THREE.Mesh(ringGeometry, ringMaterial);
  // ring.rotateX(Math.PI / 180 * 90);
  // ring.rotateX(Math.PI / 180 * 9);
  // scene.add(ring);

  //bg sphere
  const bgGeometry = new THREE.SphereGeometry(50, 60, 60);
  const bgMaterial = new THREE.MeshLambertMaterial({
    side: THREE.BackSide,
    map: new THREE.TextureLoader().load("/8k_stars.jpg")
  })
  const bgSphere = new THREE.Mesh(bgGeometry, bgMaterial);
  scene.add(bgSphere);

  //light
  const light = new THREE.PointLight(0xffffff, 5000);
  light.position.set(20, 20, 20);
  scene.add(light);

  //camera
  camera = new THREE.PerspectiveCamera(45, aspect.value, 0.1, 100);
  scene.add(camera);
  camera.add(light);
  camera.position.z = 19;

  //renderer
  const canvas = webGl.value;
  renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
  renderer.physicallyCorrectLights = true;
  renderer.setPixelRatio( window.devicePixelRatio );
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000, 0.0);
  renderer.autoClear = false;
  renderer.render(scene, camera);

  scene.add( new THREE.AmbientLight( 0xcccccc, 0.01 ) );

  const pointLight = new THREE.PointLight( 0xffffff, 1 );
  camera.add( pointLight );

  const renderScene = new RenderPass( scene, camera );

  const bloomPass = new UnrealBloomPass( new THREE.Vector2(width.value, height.value), 1, 1, 1 );
  bloomPass.threshold = params.threshold;
  bloomPass.strength = params.strength;
  bloomPass.radius = params.radius;

  composer = new EffectComposer(renderer);
  composer.setSize(window.innerWidth, window.innerHeight);
  composer.renderToScreen = true;
  composer.addPass( bloomPass );
  composer.addPass( renderScene );

  controls = new OrbitControls(camera, renderer.domElement);
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

function updateCamera() {
  camera.aspect = aspect.value;
  camera.updateProjectionMatrix();
}

function updateRender() {
  renderer.setSize(width.value, height.value);
  camera.layers.set(1);
  composer.render();
}

function animate() {
  controls.update();
  // renderer.render(scene, camera);
  composer.render();
  requestAnimationFrame(animate);
}

watch(aspect, (value) => {
  if(value) {
    updateCamera();
    updateRender();
  }
})

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
