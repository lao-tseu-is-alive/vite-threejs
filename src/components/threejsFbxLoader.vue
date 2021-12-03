<script setup>
import { onMounted, ref } from 'vue'
import {
    AnimationMixer,
    Clock,
    Color,
    Fog,
    HemisphereLight,
    DirectionalLight,
    Scene,
    PerspectiveCamera,
    CameraHelper,
    WebGLRenderer,
    GridHelper,
    BoxGeometry,
    PlaneGeometry,
    Mesh,
    MeshNormalMaterial,
    MeshPhongMaterial,

} from 'three';

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { FBXLoader } from 'three/examples/jsm/loaders/FBXLoader.js';

const CANVAS_WIDTH = 1024;
const CANVAS_HEIGHT = 768;
const clock = new Clock();
var mixer = null;

const scene = new Scene();
//scene.background = new Color(0x0000a0);
//scene.fog = new Fog(0xa0a0a0, 200, 1000);

const camera = new PerspectiveCamera(50, CANVAS_WIDTH / CANVAS_HEIGHT, 0.1, 1250000);
camera.position.set(0, 500, 500);

const renderer = new WebGLRenderer({ antialias: true });
renderer.setSize(CANVAS_WIDTH, CANVAS_HEIGHT);

const controls = new OrbitControls(camera, renderer.domElement);






const div3d = ref(null)




defineProps({
    msg: String
})

onMounted(() => {

    console.log(div3d.value);
    div3d.value.appendChild(renderer.domElement);
    const hemiLight = new HemisphereLight(0xffffff, 0x444444);
    hemiLight.position.set(4, 1, 210);
    scene.add(hemiLight);

    const dirLight = new DirectionalLight(0xffffff);
    dirLight.position.set(0, 200, 100);
    dirLight.castShadow = true;
    dirLight.shadow.camera.top = 180;
    dirLight.shadow.camera.bottom = - 100;
    dirLight.shadow.camera.left = - 120;
    dirLight.shadow.camera.right = 120;
    scene.add(dirLight);

    //scene.add(new CameraHelper(dirLight.shadow.camera));

    // ground
    //const mesh = new Mesh(new PlaneGeometry(2000, 2000), new MeshPhongMaterial({ color: 0x999999, depthWrite: false }));
    //mesh.rotation.x = - Math.PI / 2;
    //mesh.receiveShadow = true;
    //scene.add(mesh);

    const grid = new GridHelper(2000, 20, 0xffff00, 0x000000);
    grid.material.opacity = 0.15;
    grid.material.transparent = true;
    scene.add(grid);

    const geometry = new BoxGeometry(10, 10, 10);
    const material = new MeshPhongMaterial({ color: 0x000033 });
    const cube = new Mesh(geometry, material);
    scene.add(cube);

    // model
    const loader = new FBXLoader();
    loader.load('models/lausanne.fbx', function (object) {

        mixer = new AnimationMixer(object);

        //const action = mixer.clipAction(object.animations[0]);
        //action.play();

        object.traverse(function (child) {

            if (child.isMesh) {

                //child.castShadow = true;
                //child.receiveShadow = true;

            }

            if (child.isLight) {
                console.log(child)
                if (child.name == "Light") {
                    child.visible = false;                
                }            
            }

        });
        scene.add(object);
    });

    const animate = function (time) {
        requestAnimationFrame(animate);
        const delta = clock.getDelta();
        if (mixer) mixer.update(delta);
        //cube.rotation.x += 0.01;
        cube.rotation.y = time / 1000;
        renderer.render(scene, camera);
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
