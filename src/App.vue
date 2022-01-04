<style type="scss">
@import 'components/skeleton.css';
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}
img {
  width: 50%;
  max-height: 42px;
}



</style>

<script setup>
// This starter template is using Vue 3 <script setup> SFCs
// Check out https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup
import fbx from './components/threejsFbxLoader.vue'

</script>

<template>
  <div class="container">
    <!-- columns should be the immediate child of a .row -->
    <div class="row">
      <div class="twelve columns">
        <img alt="Vue logo" src="./assets/logo-lausanne.svg" />
      </div>
    </div>
    <div class="row">
      <fbx msg="Cadastre 3d Demo"/>
    </div>
  </div>
</template>

