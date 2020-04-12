<template>
  <section class="container">
    <div class="login" @click="doLogin">[login:{{user_name}}]</div>
    <div>
      <h1 class="title">{{title}}</h1>
      <p class="message">{{message}}</p>
    </div>
    <div>
      <table>
        <tr>
          <th>Message</th>
          <td>
            <input type="text" v-model="msg" />
          </td>
          <td>
            <button @click="addData">投稿</button>
          </td>
        </tr>
      </table>
    </div>
    <hr />
    <div v-if="logined">
      <ul v-for="(data,key) in json_data" :key="key">
        <li>
          <div class="list1">{{data.msg}}</div>
          <div class="list2">{{data.user}}({{data.posted}})</div>
        </li>
      </ul>
    </div>
  </section>
</template>

<script>
import firebase from "firebase/app";
import "@firebase/auth";
import "@firebase/database";
import axios from "@nuxtjs/axios";
import Logo from "~/components/Logo.vue";

let firebaseConfig = {
  apiKey: "AIzaSyBPLapP5QF_ywF9I6TokzFIcIrPLK5NdJs",
  authDomain: "nuxt-firebase-sample-e3b55.firebaseapp.com",
  databaseURL: "https://nuxt-firebase-sample-e3b55.firebaseio.com",
  projectId: "nuxt-firebase-sample-e3b55",
  storageBucket: "nuxt-firebase-sample-e3b55.appspot.com",
  messagingSenderId: "455763370080",
  appId: "1:455763370080:web:b1d71aea91b2d531546305",
  measurementId: "G-HY8XGHK3K4"
};
// Initialize Firebase
if (!firebase.apps.length) {
  firebase.initializeApp(firebaseConfig);
}
// !firebase.apps.length ? firebase.initializeApp(config) : firebase.app();
// firebase.analytics();

export default {
  components: {
    Logo
  },
  data: function() {
    return {
      title: "Board",
      message: "ミニ伝言板。最新の投稿を表示します。",
      user_name: "",
      msg: "",
      num_per_page: 10,
      json_data: {},
      logined: false
    };
  },
  methods: {
    login: function() {
      let provider = new firebase.auth.GoogleAuthProvider();
      let self = this;
      firebase
        .auth()
        .signInWithPopup(provider)
        .then(function(result) {
          console.log(result.user);
          self.user_name = result.user.displayName;
          self.message = "ログインしました。";
          self.logined = true;

          let db = firebase.database();
          let ref = db.ref("board");
          ref
            .orderByKey()
            .limitToLast(self.num_per_page)
            .on("value", function(snapshot) {
              if (firebase.auth().currentUser != null) {
                let arr = [];
                let data = snapshot.val();
                for (let item in data) {
                  arr.unshift(data[item]);
                }
                console.log(arr);
                self.json_data = arr;
              } else {
                self.json_data = {};
              }
            });
        });
    },
    logout: function() {
      firebase.auth().signOut();
      this.user_name = "";
      this.message = "ログアウトしました。";
      this.logined = false;
      this.json_data = {};
      console.log(firebase.auth());
    },
    doLogin: function() {
      console.log(firebase.auth().currentUser);

      if (firebase.auth().currentUser == null) {
        this.login();
      } else {
        this.logout();
      }
    },
    addData: function(event) {
      console.log(firebase.auth());
      if (firebase.auth().currentUser == null) {
        alert("ログインしないと投稿できません。");
        return;
      }
      let db = firebase.database();
      let user = firebase.auth().currentUser;
      let ref = db.ref("board");
      let self = this;
      let d = new Date();
      let dstr =
        d.getFullYear() +
        "-" +
        (d.getMonth() + 1) +
        "-" +
        d.getDate() +
        " " +
        d.getHours() +
        ":" +
        d.getMinutes() +
        ":" +
        d.getSeconds();
      let id = d.getTime();
      let data = {
        msg: this.msg,
        user: user.displayName,
        posted: dstr
      };
      firebase
        .database()
        .ref("board/" + id)
        .set(data);
      this.msg = "";
      this.message = "投稿しました。";
    }
  },
  created: function() {
    if (firebase.auth().currentUser == null) {
      this.login();
    }
    console.log(firebase.auth().currentUser);
  }
};
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  /* align-items: center; */
  /* text-align: center; */
  flex-direction: column;
  max-width: 80%;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
hr {
  margin: 10px 0px;
}
input {
  font-size: 14pt;
}
button {
  font-size: 14pt;
}
.list1 {
  text-align: left;
  font-size: 16pt;
}
.list2 {
  text-align: right;
  font-size: 10pt;
}
.login {
  font-size: 12pt;
  font-weight: 600;
  cursor: pointer;
}
.message {
  font-size: 16pt;
}
ul {
  background-color: aliceblue;
}
li {
  font-size: 14pt;
  padding: 10px;
}
</style>
