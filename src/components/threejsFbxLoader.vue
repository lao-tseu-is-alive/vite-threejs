<style type="scss">
a {
  color: #42b983;
}

.cglog {
    text-align: left;
    background-color: #2c3e50;
    color: #9effff;
}

pre {
  margin: 0.2rem;
  margin-bottom: 0.1rem !important;
  font-family: 'Lucida Console', 'Roboto Mono', monospace;
  font-size: 1rem;
}

</style>

<template>
  <div class="row" ref="mainRow">
    <div class="nine columns">
      <div ref="div3d"></div>
    </div>
    <div class="three columns">
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
import {isNullOrUndefined} from "cgil-html-utils";

const CANVAS_WIDTH = () => Math.max(Math.round((Math.min(window.innerWidth,screen.availWidth) - 100)*0.76), 300) ;
const CANVAS_HEIGHT = () => (Math.min(window.innerHeight,screen.availHeight) - 100);
const clock = new Clock();
let mixer = null;
let loadProgress = 0;

const scene = new Scene();
//scene.background = new Color(0x0000a0);
//scene.fog = new Fog(0xa0a0a0, 200, 1000);

const camera = new PerspectiveCamera(
    50,
    CANVAS_WIDTH() / CANVAS_HEIGHT(), 0.1, 1250000);
camera.position.set(-55000, 25500, 18000);

const renderer = new WebGLRenderer({antialias: true});
renderer.setSize(CANVAS_WIDTH(), CANVAS_HEIGHT());

const controls = new OrbitControls(camera, renderer.domElement);
controls.addEventListener('change', function () {
  console.log(
      `distance : ${controls.getDistance()}, camera position (x:${camera.position.x}, y:${camera.position.y}, z:${camera.position.z}`,
      camera);
});
const div3d = ref(null);
const divLog = ref(null);
const mainRow = ref(null);


const props = defineProps({
  fbxModel: {
    type: String,
    default: 'models/fiches.fbx',
  }
})

onMounted(() => {
  console.warn('## onMounted threejsFbxLoader.vue')
  const startTime = performance.now()

  const cgLog = function (msg, ...args) {
    const endTime = performance.now()
    const timeLapse = (endTime - startTime).toFixed(0);
    if (!isNullOrUndefined(divLog.value)) {
      divLog.value.innerHTML = `<pre>[${timeLapse}㎳]: ${msg}</pre> ${divLog.value.innerHTML}`;
    }
    console.log(msg);
    if (args.length > 0) {
      args.map((v) => console.log(v))
    }
  }

  div3d.value.appendChild(renderer.domElement);
  const hemiLight = new HemisphereLight(0xffffff, 0xaabbaa, 0.80);
  hemiLight.position.set(0, 20000, 0);
  scene.add(hemiLight);
  const hemiLightHelper = new HemisphereLightHelper(hemiLight, 1000);
  scene.add(hemiLightHelper);

  const dirLight = new DirectionalLight(0xffffff, 0.2);
  dirLight.position.set(-80500, 40500, 65000);
  scene.add(dirLight);
  const dirLightHelper = new DirectionalLightHelper(dirLight, 1000);
  scene.add(dirLightHelper);

  const geometry = new CylinderGeometry(500, 500, 10000);
  const material = new MeshPhongMaterial({color: 0x000033});
  const axe = new Mesh(geometry, material);
  //scene.add(axe);

  // model
  const loader = new FBXLoader();
  cgLog(`will try to load : ${props.fbxModel}`);
  loader.load(
      props.fbxModel,
      // called when the resource is loaded
      function (object) {
        cgLog('# FBX Object loaded !')
        //onResize();
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
          cgLog(currentLoad + ' % loaded');
          loadProgress = currentLoad;
        }
      },
      // called when loading has errors
      function (error) {
        cgLog(`## An ERROR happened loading : ${props.fbxModel}. ERROR : ${error} `, error);
      }
  ); // end of fbx FBXLoader.load()


  const onResize = function () {
    let width = CANVAS_WIDTH();
    let height = CANVAS_HEIGHT();
    if (!isNullOrUndefined(div3d.value)) {
      width = div3d.value.offsetWidth - 2;
      //height = div3d.value.offsetHeight < 600 ? 598 : div3d.value.offsetHeight - 2;
    }
    //if (height < 600 ) height = 550;
    //if (height > (Math.min(window.innerHeight,screen.availHeight) - 300)) height = CANVAS_HEIGHT;
    camera.aspect = width / height;
    cgLog(`@@onResize w x h : ${width} x ${height}`);
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
  //onResize();
  window.addEventListener('resize', onResize);
  animate();
})


const count = ref(0)


</script>

