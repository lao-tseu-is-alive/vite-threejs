<style type="scss">
@import 'components/skeleton.css';
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 3px;
}
img {
  width: 50%;
  max-height: 42px;
}
</style>

<template>
    <div class="row">
      <div class="twelve columns">
        <img alt="Vue logo" src="./assets/logo-lausanne.svg" />
      </div>
    </div>
    <fbx :fbx-model="fbxModel"/>
</template>


<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
//this.$el.parentElement
import {onMounted} from 'vue'
import fbx from './components/threejsFbxLoader.vue'
import { getStringParam, getEl, addClass } from 'cgil-html-utils';
let fbxModel = getStringParam('model', 'fiches');

if (fbxModel == 'fiches') {
  fbxModel = 'models/fiches.fbx';
} else {
  fbxModel = 'models/lausanne.fbx';
}


onMounted(() => {
  console.warn('## onMounted App.vue');
  // const myApp = getEl('app');
  addClass('app', 'container');
})
</script>

