<script setup>
import { onMounted, ref } from 'vue'
import { 
  Scene,
  PerspectiveCamera,
  WebGLRenderer,
  BoxGeometry,
  Mesh,
  MeshNormalMaterial,
  } from 'three';

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
//import { OrbitControls } from './jsm/controls/OrbitControls.js';
//import { FBXLoader } from './jsm/loaders/FBXLoader.js';

const CANVAS_WIDTH = 1024;
const CANVAS_HEIGHT = 768;
const scene = new Scene();
const camera = new PerspectiveCamera(50, CANVAS_WIDTH / CANVAS_HEIGHT, 1, 1000 );
const renderer = new WebGLRenderer({ antialias: true });
renderer.setSize( CANVAS_WIDTH, CANVAS_HEIGHT );

const controls = new OrbitControls(camera, renderer.domElement);

const div3d = ref(null)




defineProps({
  msg: String
})

onMounted(()=>{
  
  console.log(div3d.value);
  div3d.value.appendChild( renderer.domElement );
  const geometry = new BoxGeometry( 1, 1, 1 );
  const material = new MeshNormalMaterial( { color: 0x00ff00 } );
  const  cube = new Mesh( geometry, material );
  scene.add( cube );
  camera.position.z = 5;
  
  const animate = function (time) {
    requestAnimationFrame( animate );
    //cube.rotation.x += 0.01;
    cube.rotation.y = time / 1000;
    renderer.render( scene, camera );
  };
  animate();


})


const count = ref(0)


</script>

<template>
  <h1>{{ msg }}</h1>
  <div ref="div3d" style="text-align: center;"></div>
  
  
</template>

<style scoped>
a {
  color: #42b983;
}

</style>
