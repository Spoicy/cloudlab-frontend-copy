<template>
  <div class="">
    <h1>Login</h1>
    <LoginComponent @loginSubmit="login" />
  </div>
</template>

<script>
import { CognitoIdentityProviderClient, InitiateAuthCommand } from "@aws-sdk/client-cognito-identity-provider";
import LoginComponent from "../components/LoginComponent.vue";
import config from "../assets/config.json";

export default {
  components: {
    LoginComponent
  },
  data() {
    return {
      cognitoClient: new CognitoIdentityProviderClient({
        region: config.region,
      })
    }
  },
  methods: {
    async login(username, password) {
      const params = {
        AuthFlow: "USER_PASSWORD_AUTH",
        ClientId: config.clientId,
        AuthParameters: {
          USERNAME: username,
          PASSWORD: password,
        }
      }
      try {
        console.log("trying log in")
        console.log(params)
        const command = new InitiateAuthCommand(params);
        const { AuthenticationResult } = await this.cognitoClient.send(command);
        if (AuthenticationResult) {
          sessionStorage.setItem("idToken", AuthenticationResult.IdToken || '');
          sessionStorage.setItem("accessToken", AuthenticationResult.AccessToken || '');
          sessionStorage.setItem("refreshToken", AuthenticationResult.RefreshToken || '');
          sessionStorage.setItem("username", username || '');
          this.$router.push('/');
          return AuthenticationResult;
        }
      } catch (error) {
        console.error("Error signing in: ", error);
        throw error;
      }
    }
  },
}
</script>

<style lang="scss" scoped>

</style>