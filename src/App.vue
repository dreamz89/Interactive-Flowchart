<template>
  <div id="app">
    <InteractiveFlowchart></InteractiveFlowchart>
  </div>
</template>

<script>
import InteractiveFlowchart from './components/InteractiveFlowchart.vue'

export default {
  name: 'app',
  components: { InteractiveFlowchart }
}
</script>

<style>
#app {
  height: 100vh;
  width: 100vw;
}
</style>
