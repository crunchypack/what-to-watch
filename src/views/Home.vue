<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <MainPage></MainPage>
  </div>
</template>

<script>
// @ is an alias to /src
import MainPage from "@/components/MainPage.vue";

export default {
  name: "home",
  components: {
    MainPage
  }
};
</script>
