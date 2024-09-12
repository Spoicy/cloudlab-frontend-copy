<script>
import { RouterLink, RouterView } from 'vue-router'

export default {
  beforeMount() {
    const accessToken = sessionStorage.getItem("accessToken");
    if (accessToken == undefined || accessToken == '' || accessToken == null) {
      this.$router.push('/login');
    }
  }
}
</script>

<template>
  <RouterView />
</template>

<style>
#app {
  text-align: center;
  width: 50%;
  margin: auto;
}
</style>
