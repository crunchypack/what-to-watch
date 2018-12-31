<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link>|
      <router-link to="/about">About</router-link>
    </div>
    <div class="logo">
      <img alt="WTW logo" src="./assets/logo.png">
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
@media screen and (max-width: 600px) {
  .logo img {
    width: 100%;
    margin: 0;
  }
}
</style>
