<template>
  <v-container pa-12 fill-height>
    <v-layout align-center text-center wrap>
      <v-flex xs12>
        <v-alert v-model="alert" type="error" dismissible>{{errorMessage}}</v-alert>
        <v-alert v-model="success" type="success" dismissible>SUCCESS!</v-alert>
        <v-img :src="require('../assets/thai monk.svg')" class="my-3" contain height="200"></v-img>
        <h1>「Devotter」</h1>
        <h2>
          その精進
          <br />ツイートしませんか
        </h2>
        <br />
        <h5>このアプリケーションは、1日1回AtCoderでのAC数をTwitterにツイートしてくれるアプリケーションです。</h5>
        <br />
        <v-form>
          <v-text-field
            @change="changeField"
            type="text"
            v-model="atcoderId"
            v-if="login"
            label="AtCoderID"
            color="#00acee"
            required
          ></v-text-field>
        </v-form>
        <v-btn
          @click="signin"
          v-bind:disabled="isnull"
          v-show="login"
          color="#00acee"
          rounded
        >Sign In To Twitter</v-btn>
        <v-btn @click="logout" v-show="!login" color="#00acee" rounded>Log Out from Twitter</v-btn>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
import firebase from "firebase";
import axios from "axios";

export default {
  name: "guestUserScreen",
  methods: {
    logout: function() {
      firebase
        .auth()
        .signOut()
        .then(()=>{
          this.success=true;
          this.login=true;
        })
        .catch(()=>{
          this.alert=true;
          this.errorMessage="サインアウトに失敗しました。"
        });
    },
    signin: function() {
      var db = firebase.firestore();
      var document = this.atcoderId;
      axios
        .get(this.userApi + this.atcoderId)
        .then(response => {
          this.userInfo = response;
          const provider = new firebase.auth.TwitterAuthProvider();
          this.success = true;
          firebase
            .auth()
            .signInWithPopup(provider)
            .then(function(result) {
              let token = result.credential.accessToken;
              let secret = result.credential.secret;
              db.collection("users")
                .doc(document)
                .set({
                  accessToken: token,
                  accessTokenSecret: secret
                });
            })
            .catch(function() {
              this.alert = true;
              this.errorMessage = "ERROR!";
            });
        })
        .catch(() => {
          this.alert = true;
          this.errorMessage = "無効なAtCoderID名です。";
        });
    },
    changeField: function() {
      this.atcoderId ? (this.isnull = false) : (this.isnull = true);
    }
  },
  created: function() {
    firebase.auth().onAuthStateChanged(user => {
      if (user) {
        this.login = false;
      }
    });
  },
  data: function() {
    return {
      atcoderId: "",
      isnull: true,
      userInfo: "",
      userApi: "https://kenkoooo.com/atcoder/atcoder-api/v2/user_info?user=",
      alert: false,
      errorMessage: "",
      success: false,
      login: true
    };
  }
};
</script>
