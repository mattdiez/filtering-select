<template>
  
  <div id="app">
    <link rel="stylesheet" href="https://unpkg.com/bootstrap@4.1.3/dist/css/bootstrap.min.css"/>
    <img alt="Vue logo" src="./assets/logo.png">

    <div class="container">
      <div class="row">
        <div class="form-group">
          <label class="form-label">FilteringSelect Test</label>
          <FilteringSelect name="zip" placeHolder="Override a placeHolder"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import FilteringSelect from './components/FilteringSelect.vue'

export default {
  name: 'app',
  components: {
    HelloWorld,
    FilteringSelect
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
