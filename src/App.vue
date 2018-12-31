<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link>|
      <router-link to="/about">About</router-link>
    </div>
    <div class="logo">
      <img alt="Vue logo" src="./assets/logo.png">
    </div>
    <router-view/>
  </div>
</template>

<style lang="scss">
#nav {
  text-align: center;
}
.logo img {
  display: block;
  margin: auto;
}
</style>
