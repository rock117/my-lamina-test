<template>
  <div class="three">
    <div>
      <div class="el-row">
        <div class="el-col-2">position:</div>
        <div class="el-col-2">
          <el-input @change="v => onPosChanged('posX', v)" v-model="posX" placeholder="x"/>
        </div>
        <div class="el-col-2">
          <el-input @change="v => onPosChanged('posY', v)" v-model="posY" placeholder="y"/>
        </div>
        <div class="el-col-2">
          <el-input @change="v => onPosChanged('posZ', v)" v-model="posZ" placeholder="z"/>
        </div>
      </div>
      <div class="el-row">
        <div class="el-col-2">origin:</div>
        <div class="el-col-2">
          <el-input @change="v => onOriginChanged('oX', v)" v-model="originX" placeholder="x"/>
        </div>
        <div class="el-col-2">
          <el-input @change="v => onOriginChanged('oY', v)" v-model="originY" placeholder="y"/>
        </div>
        <div class="el-col-2">
          <el-input @change="v => onOriginChanged('oZ', v)" v-model="originZ" placeholder="z"/>
        </div>
      </div>

      <div class="el-row">
        <div class="el-col-2">Near Far:</div>
        <div class="el-col-2">
          <el-input @change="v => onNearFarChanged('near', v)" v-model="near" placeholder="near"/>
        </div>
        <div class="el-col-2">
          <el-input @change="v => onNearFarChanged('far', v)" v-model="far" placeholder="far"/>
        </div>
      </div>
      <div class="el-row">
        <div class="el-col-2">mapping:</div>
        <div class="el-col-2">
          <el-select v-model="mapping" @change="onMappingChanged" class="m-2" placeholder="Select" size="large">
            <el-option
                v-for="item in mappings"
                :key="item"
                :label="item"
                :value="item"
            />
          </el-select>
        </div>
      </div>
    </div>
    <div id="ws">

    </div>
  </div>
</template>
<script setup>
import * as THREE from "three";
import {ref, onMounted} from "vue"
import {LayerMaterial, Depth} from "lamina/vanilla";
import {OrbitControls} from 'three/addons/controls/OrbitControls.js';

let camera, controls, scene, renderer, mesh;
const posX = ref(1)
const posY = ref(1)
const posZ = ref(1)

const originX = ref(0)
const originY = ref(0)
const originZ = ref(0)

const near = ref(0)
const far = ref(2)
const mapping = ref("world")
const mappings = ref([
  "vector",
  "world",
  "camera"
])

const onPosChanged = (key, v) => {
  if (key == "posX") {
    posX.value = parseFloat(v)
    mesh.position.set(posX.value, posY.value, posZ.value)
  } else if (key == "posY") {
    posY.value = parseFloat(v)
    mesh.position.set(posX.value, posY.value, posZ.value)

  } else if (key == "posZ") {
    posZ.value = parseFloat(v)
    mesh.position.set(posX.value, posY.value, posZ.value)
  }
  refreshMaterial()
}
const onOriginChanged = (key, v) => {
  if (key == "oX") {
    originX.value = parseFloat(v)
    update("origin", new THREE.Vector3(originX.value, originY.value, originZ.value))
  } else if (key == "oY") {
    originY.value = parseFloat(v)
    update("origin", new THREE.Vector3(originX.value, originY.value, originZ.value))
  } else if (key == "oZ") {
    originZ.value = parseFloat(v)
    update("origin", new THREE.Vector3(originX.value, originY.value, originZ.value))
  }
}
const onNearFarChanged = (key, v) => {
  if (key == "near") {
    near.value = parseFloat(v)
    update("near", near.value)
  } else if (key == "far") {
    far.value = parseFloat(v)
    update("far", far.value)
  }
}
const onMappingChanged = v => {
  console.log("you select " + v)
  mapping.value = v
  update("mapping", v)
}
const refreshMaterial = () => {
  const material = mesh.material
  const layer = mesh.material.layers[0]
  layer.buildShaders(Depth);
  material.refresh();
  material.needsUpdate = true;
  mesh.updateMatrix()
  mesh.updateMatrixWorld()
  renderer.clear();
  renderer.render(scene, camera);
}
const update = (key, value) => { // key: origin, mapping, near, far ...etc
  const material = mesh.material
  const layer = mesh.material.layers[0]
  layer[key] = value
  material["layers"] = [layer];
  layer.buildShaders(Depth);
  material.refresh();
  material.needsUpdate = true;

  renderer.clear();
  renderer.render(scene, camera);

}
const doRender = () => {
  renderer.render(scene, camera)
}
onMounted(() => {
  start()
})

function init() {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0xcccccc);
  const geometry = new THREE.BoxGeometry(1, 2, 3)

  let material = new LayerMaterial({
    color: '#d9d9d9',
    lighting: 'phong',
    transmission: 1,
    layers: [
      new Depth({
        colorA: 'red',
        colorB: 'blue',
        alpha: 1,
        mode: 'normal',
        near: near.value,
        mapping: mapping.value,
        far: far.value,
        origin: new THREE.Vector3(originX.value, originY.value, originZ.value),
      }),
    ],
  })
  console.log("material mapping", material.layers[0].mapping)
  material.layers[0].mapping = mapping.value
  console.log("material mapping", material.layers[0].mapping)
  mesh = new THREE.Mesh(geometry, material);
  mesh.position.x = posX.value;
  mesh.position.y = posY.value;
  mesh.position.z = posZ.value;
  scene.add(mesh);

  const directionalLight = new THREE.DirectionalLight();
  directionalLight.position.x = 6;
  directionalLight.position.y = 4;
  directionalLight.position.z = 2;
  scene.add(directionalLight);

  const amlight = new THREE.AmbientLight(0x404040); // soft white light scene.add( light );
  scene.add(amlight);
  const gridHelper = new THREE.GridHelper(100, 10);
  scene.add(gridHelper);

  // create a camera, which defines where we're looking at.
  camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
  camera.position.x = 4
  camera.position.y = 4
  camera.position.z = 4
  camera.lookAt(scene.position);

  renderer = new THREE.WebGLRenderer();

  controls = new OrbitControls(camera, renderer.domElement);
  controls.listenToKeyEvents(window); // optional

  //controls.addEventListener( 'change', render ); // call this only in static scenes (i.e., if there is no animation loop)

  controls.enableDamping = true; // an animation loop is required when either damping or auto-rotation are enabled
  controls.dampingFactor = 0.05;

  controls.screenSpacePanning = false;

  // controls.minDistance = 100;
  // controls.maxDistance = 500;

  controls.maxPolarAngle = Math.PI / 2;


  // renderer.setClearColorHex();
  //  renderer.setClearColor();
  renderer.setSize(window.innerWidth, window.innerHeight);

  document.getElementById("ws").appendChild(renderer.domElement);
  renderer.render(scene, camera)
}

function animate() {

  requestAnimationFrame(animate);

  controls.update(); // only required if controls.enableDamping = true, or if controls.autoRotate = true

  render();

}

function render() {

  renderer.render(scene, camera);

}

function start() {
  init()
  animate()
}

</script>
<style>

</style>