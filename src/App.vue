<template>
  <div id="app">
    <UserInput/>
  </div>
</template>

<script>
import UserInput from './components/input/user-input'
export default {
  name: 'App',
  components: {
    UserInput,
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
