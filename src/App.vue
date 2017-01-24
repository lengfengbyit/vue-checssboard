<template>
  <div id="app">
    <chessboard></chessboard>
  </div>
</template>

<script>
import Chessboard from './components/chessboard'

export default {
  name: 'app',
  components: {
    Chessboard
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
