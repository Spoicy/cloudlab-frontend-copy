<template>
  <div>
    <div class="mb-3">
      <label for="username" class="form-label">Username</label>
      <input v-model="username" class="form-control" type="text" name="username" id="username">
    </div>
    <div class="mb-3">
      <label for="username" class="form-label">Password</label>
      <input v-model="password" class="form-control" type="password" name="password" id="password">
    </div>
    <button @click="submitLogin" class="btn btn-primary">Login</button>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        username: "",
        password: ""
      }
    },
    methods: {
      submitLogin() {
        console.log(this.username, this.password);
        this.$emit("loginSubmit", this.username, this.password);
      }
    },
  }
</script>

<style lang="scss" scoped>

</style>