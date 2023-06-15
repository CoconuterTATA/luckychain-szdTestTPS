<template>
  <div id="app">
    <EChartsLineChart />
  </div>
</template>

<script>
import EChartsLineChart from './components/TpsTest.vue'

export default {
  components: {
    EChartsLineChart
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
