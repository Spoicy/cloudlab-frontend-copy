<script>
import config from "../assets/config.json";

  export default {
    data() {
      return {
        file: null,
        categoryAnswer: null,
        category: "",
        overallYes: 0,
        overallNo: 0,
        userYes: 0,
        userNo: 0,
        imageUrl: "",
        email: "",
        username: ""
      }
    },
    methods: {
      async uploadFile() {
        console.log(this.file)
        const fileName = this.file.name;
        let fileSplit = fileName.split('.');
        const newFileName = fileSplit[0] + Math.floor(Date.now() / 1000) + "." + fileSplit[1];
        const response = await fetch(config.uploadApiUrl, {
          method: 'PUT',
          body: JSON.stringify({username: this.username, filename: newFileName})
        });
        if (response == null) {
          return;
        }
        const responseJson = await response.json();
        const uploadUrl = responseJson.uploadUrl;
        const storageUrl = responseJson.storageUrl;
        let storageSplit = storageUrl.split('.us-east-1');
        const newStorageUrl = storageSplit[0] + storageSplit[1];
        this.imageUrl = newStorageUrl;
        console.log(responseJson, uploadUrl, storageUrl)

        await fetch(uploadUrl, {
          method: 'PUT',
          body: this.file,
          headers: {
            'Content-Type': this.file.type
          }
        });
        
        const responseRecog = await fetch("https://xtq3s21nkg.execute-api.us-east-1.amazonaws.com/prod/recognition", {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: 
            JSON.stringify({
              'imageUrl': newStorageUrl,
              'metadata': {
                'user': this.email,
              }
            }
          )
        });
        console.log("fetch done")
        const recogJson = await responseRecog.json();
        const recogBody = JSON.parse(recogJson.body)
        this.category = recogBody.category;
        console.log(this.category, recogBody);
      },
      fileChange(e) {
        var files = e.target.files;
        if (!files.length) {
          return;
        }
        this.file = files[0];
        console.log(this.file)
      },
      async submitRating() {
        const response = await fetch("https://xtq3s21nkg.execute-api.us-east-1.amazonaws.com/prod/save-response", {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body:
            JSON.stringify({
              'imageUrl': this.imageUrl,
              'metadata': {
                'user': this.email,
                'recognized': this.category,
                'correct': this.categoryAnswer
              }
            }
          )
        });
        const responseJson = await response.json();
        console.log(responseJson);
        console.log("rating submitted");
        this.categoryAnswer = null;
        this.category = "";
        this.sumStats();
      },
      async sumStats() {
        const response = await fetch("https://xtq3s21nkg.execute-api.us-east-1.amazonaws.com/prod/stats");
        const responseJson = await response.json();
        const responseBody = JSON.parse(responseJson.body);
        let overallYesTotal = 0;
        let overallNoTotal = 0;
        Object.keys(responseBody.summary).forEach(function(key,index) {
          const cat = responseBody.summary[key];
          overallYesTotal += cat.correct;
          overallNoTotal += (cat.total - cat.correct);
        });
        this.overallYes = overallYesTotal;
        this.overallNo = overallNoTotal;
        const responseUser = await fetch("https://xtq3s21nkg.execute-api.us-east-1.amazonaws.com/prod/stats-user", {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body:
            JSON.stringify({
              "user": this.email
            }
          )
        });
        const responseUserJson = await responseUser.json();
        const responseUserBody = JSON.parse(responseUserJson.body);
        let userYesTotal = 0;
        let userNoTotal = 0;
        Object.keys(responseUserBody.summary).forEach(function(key,index) {
          const cat = responseUserBody.summary[key];
          userYesTotal += cat.correct;
          userNoTotal += (cat.total - cat.correct);
        });
        this.userYes = userYesTotal;
        this.userNo = userNoTotal;
        console.log('this ran without problem.', responseBody, responseUserBody);
      },
      async setEmail() {
        const responseUser = await fetch(config.userEmailApiUrl, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body:
            JSON.stringify({
              "username": sessionStorage.getItem('username')
            }
          )
        });
        const responseUserJson = await responseUser.json();
        this.email = responseUserJson.email;
        console.log(this.email);
      },
      logout() {
        sessionStorage.removeItem("idToken");
        sessionStorage.removeItem("accessToken");
        sessionStorage.removeItem("refreshToken");
        sessionStorage.removeItem("username");
        this.$router.push('/login');
      },
      isAuthenticated() {
        const accessToken = sessionStorage.getItem("accessToken");
        if (accessToken !== '' && accessToken !== null) {
          return true;
        }
        return false;
        /*const accessToken = session("accessToken");
        console.log(accessToken);
        return !!accessToken;*/
      }
    },
    mounted() {
      this.setEmail();
      this.sumStats();
      this.username = sessionStorage.getItem("username");
    }
  }
</script>

<template>
  <main v-if="isAuthenticated()">
    <div class="w-50 m-auto pt-4">
      <h2 class="pb-2">Upload your image</h2>
      <input class="form-control" @change="fileChange" type="file" id="image" name="image">
      <button class="btn btn-primary mt-2" @click="uploadFile">Submit</button>
    </div>
    <div v-show="category != ''" class="pt-4">
      <h2>Detected category</h2>
      <p>Was the detected category correct?</p>
      <p class="fw-bold">Category: {{ category }}</p>
      <div class="d-inline-flex align-items-center gap-2">
        <div class="form-check">
          <input class="form-check-input" v-model="categoryAnswer" type="radio" name="evaluation" id="correct" value="yes">
          <label class="form-check-label" for="correct">Yes</label>
        </div>
        <div class="form-check">
          <input class="form-check-input" v-model="categoryAnswer" type="radio" name="evaluation" id="incorrect" value="no">
          <label class="form-check-label" for="correct">No</label>
        </div>
      </div>
      <br>
      <button class="btn btn-primary mt-2" @click="submitRating">Submit rating</button>
    </div>
    <div class="w-50 m-auto pt-4">
      <h2>Stats</h2>
      <div class="row">
        <h3>Overall</h3>
        <div class="col-6">
          <h4>Yes</h4>
          <p class="fw-bold">{{ overallYes }}</p>
        </div>
        <div class="col-6">
          <h4>No</h4>
          <p class="fw-bold">{{ overallNo }}</p>
        </div>
      </div>
      <div class="row">
        <h3>User</h3>
        <div class="col-6">
          <h4>Yes</h4>
          <p class="fw-bold">{{ userYes }}</p>
        </div>
        <div class="col-6">
          <h4>No</h4>
          <p class="fw-bold">{{ userNo }}</p>
        </div>
      </div>
    </div>
    <div>
      <button @click="logout" class="btn btn-secondary">Logout</button>
    </div>
  </main>
</template>
