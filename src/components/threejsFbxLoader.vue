<style scoped>
a {
  color: #42b983;
}

.cglog {
  text-align: left;
  background-color: #2c3e50;
  color: #9effff;
}

</style>

<template>
  <h2>{{ msg }}</h2>
  <div class="row">
    <div class="two-thirds column">
      <div ref="div3d"></div>
    </div>
    <div class="one-third column">
      <div ref="divLog" class="cglog"></div>
    </div>
  </div>


</template>


<script setup>
import {onMounted, ref} from 'vue'
import {
  AnimationMixer,
  Clock,
  Color,
  Fog,
  HemisphereLight,
  HemisphereLightHelper,
  DirectionalLight,
  DirectionalLightHelper,
  Scene,
  PerspectiveCamera,
  CameraHelper,
  WebGLRenderer,
  GridHelper,
  BoxGeometry,
  PlaneGeometry,
  CylinderGeometry,
  Mesh,
  MeshNormalMaterial,
  MeshPhongMaterial,

} from 'three';

import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls';
import {FBXLoader} from 'three/examples/jsm/loaders/FBXLoader.js';

const fbxModel = 'models/lausanne.fbx';

let CANVAS_WIDTH = 800;
let CANVAS_HEIGHT = 600;
const clock = new Clock();
let mixer = null;
let loadProgress = 0;

const scene = new Scene();
//scene.background = new Color(0x0000a0);
//scene.fog = new Fog(0xa0a0a0, 200, 1000);

const camera = new PerspectiveCamera(
    50,
    CANVAS_WIDTH / CANVAS_HEIGHT, 0.1, 1250000);
camera.position.set(-55000, 25500, 18000);

const renderer = new WebGLRenderer({antialias: true});
renderer.setSize(CANVAS_WIDTH, CANVAS_HEIGHT);

const controls = new OrbitControls(camera, renderer.domElement);
controls.addEventListener('change', function () {
  console.log(
      `distance : ${controls.getDistance()}, camera position (x:${camera.position.x}, y:${camera.position.y}, z:${camera.position.z}`,
      camera);
});
const div3d = ref(null)
const divLog = ref(null)


defineProps({
  msg: String
})

onMounted(() => {
  console.warn('## onMounted')
  const startTime = performance.now()

  const cgLog = function (msg, ...args) {
    const endTime = performance.now()
    const timeLapse = (endTime - startTime).toFixed(0);
    divLog.value.innerHTML = `<p>[${timeLapse}㎳]: ${msg}</p> ${divLog.value.innerHTML}`;
    console.log(msg);
    if (args.length > 0) {
      args.map((v) => console.log(v))
    }
  }

  cgLog('div3d : ', div3d.value);
  div3d.value.appendChild(renderer.domElement);
  const hemiLight = new HemisphereLight(0xffffff, 0xaabbaa, 0.80);
  hemiLight.position.set(0, 20000, 0);
  scene.add(hemiLight);
  const hemiLightHelper = new HemisphereLightHelper(hemiLight, 1000);
  scene.add(hemiLightHelper);

  const dirLight = new DirectionalLight(0xffffff, 0.2);
  dirLight.position.set(-80500, 40500, 65000);
  scene.add(dirLight);
  /*
   dirLight.castShadow = true;
   dirLight.shadow.mapSize.width = 2048;
   dirLight.shadow.mapSize.height = 2048;

   const d = 500;

   dirLight.shadow.camera.left = -d;
   dirLight.shadow.camera.right = d;
   dirLight.shadow.camera.top = d;
   dirLight.shadow.camera.bottom = -d;

   dirLight.shadow.camera.far = 3500;
   dirLight.shadow.bias = -0.0001;
   */

  const dirLightHelper = new DirectionalLightHelper(dirLight, 1000);
  scene.add(dirLightHelper);
  //scene.add(new CameraHelper(dirLight.shadow.camera));

  // ground
  //const mesh = new Mesh(new PlaneGeometry(2000, 2000), new MeshPhongMaterial({ color: 0x999999, depthWrite: false }));
  //mesh.rotation.x = - Math.PI / 2;
  //mesh.receiveShadow = true;
  //scene.add(mesh);

  /*
   const grid = new GridHelper(100000, 1000, 0xffff00, 0x000000);
   grid.material.opacity = 0.45;
   grid.material.transparent = true;
   scene.add(grid);*/

  const geometry = new CylinderGeometry(500, 500, 10000);
  const material = new MeshPhongMaterial({color: 0x000033});
  const axe = new Mesh(geometry, material);
  //scene.add(axe);

  // model
  const loader = new FBXLoader();
  loader.load(
      fbxModel,
      // called when the resource is loaded
      function (object) {
        cgLog('# FBX Object loaded !')
        mixer = new AnimationMixer(object);

        //const action = mixer.clipAction(object.animations[0]);
        //action.play();
        cgLog('# FBX Object parsing begin !')
        object.traverse(function (child) {

          if (child.isMesh) {
            child.castShadow = false;
            child.receiveShadow = false;
          }

          if (child.isLight) {
            cgLog('Found Light object : ', child)
            if (child.name == "Light") {
              child.visible = false;
            }
            // disable all other lights
            child.visible = false;
          }
        });
        cgLog('# FBX Object parsing end !')
        scene.add(object);
        cgLog('# FBX Object added to scene !')
      },
      // called while loading is progressing
      function (xhr) {
        const currentLoad = (xhr.loaded / xhr.total * 100).toFixed(1)
        if (currentLoad < 98) {
          if ((currentLoad - loadProgress) > 2.0) {
            cgLog(currentLoad + ' % loaded', xhr.loaded);
            loadProgress = currentLoad;
          }
        } else {
          cgLog(currentLoad + ' % loaded', xhr.loaded);
          loadProgress = currentLoad;
        }
      },
      // called when loading has errors
      function (error) {
        cgLog(`## An ERROR happened loading : ${fbxModel}. ERROR : ${error} `, error);
      }
  ); // end of fbx FBXLoader.load()


  const onResize = function () {
    const width = div3d.value.offsetWidth - 2;
    const height = div3d.value.offsetHeight - 2;
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
    controls.update();
  }

  const animate = function (time) {
    requestAnimationFrame(animate);
    const delta = clock.getDelta();
    if (mixer) mixer.update(delta);
    //cube.rotation.x += 0.01;
    //cube.rotation.y = time / 1000;
    renderer.render(scene, camera);

  };
  onResize();
  window.addEventListener('resize', onResize);
  animate();


})


const count = ref(0)


</script>

