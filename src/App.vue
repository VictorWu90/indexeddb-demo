<template>
  <div id="app">
    <h1>indexedDB的基本使用</h1>
    <div id="nav">
      <router-link to="/">Base</router-link> |
      <router-link to="/about">More</router-link>
    </div>
    <router-view/>
  </div>
</template>

<style>
body{
  padding: 20px !important;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
</style>
