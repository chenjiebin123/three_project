<template>
  <div id="app">
    <ThreePage />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import ThreePage from './components/demo.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    ThreePage
  }
}
</script>

<style lang="scss">
body,html {
  width: 100%;
  height: 100px;
  margin: 0;
}
#app {
  width: 100%;
  height: 100px;
}
</style>
