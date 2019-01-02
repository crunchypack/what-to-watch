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
    <div class="footer">
      <slot name="footer">
        <div>
          Site created by
          <a href="mailto:simon.wolf.lobo@gmail.com">Simon Lobo</a>
        </div>
        <div>
          Content licensed under
          <a
            href="https://creativecommons.org/licenses/by-nc/4.0/"
            target="_blank"
            rel="noopener noreferrer"
          >CC BY-NC 4.0</a>
        </div>
      </slot>
    </div>
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
.footer {
  width: 50%;
  font-size: 80%;
  margin: 0 auto;
  padding: 10%;
}
.footer div {
  width: 50%;
  margin: auto;
}
@media screen and (max-width: 600px) {
  .logo img {
    width: 100%;
    margin: 0;
  }
  .footer {
    width: 100%;
  }
  .footer div {
    width: 100%;
  }
}
</style>
