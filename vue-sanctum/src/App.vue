<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <HelloWorld msg="Welcome to Your Vue.js App" />
  </div>
</template>

<script>
import Echo from "laravel-echo";
// eslint-disable-next-line no-unused-vars
import Pusher from "pusher-js";
import axios from "axios";

axios
  .post("http://127.0.0.1:8000/api/sanctum/token", {
    email: "johndoe@example.org",
    password: "secret",
  })
  .then(({ data }) => {
    let token = data;
    //
    axios({
      method: "GET",
      url: "http://127.0.0.1:8000/api/user",
      headers: {
        Authorization: `Bearer ${token}`,
      },
    }).then(({ data }) => {
      console.log(data);

      let echo = new Echo({
        broadcaster: "pusher",
        key: "s3cr3t",
        wsHost: "127.0.0.1",
        wsPort: 6001,
        forceTLS: false,
        cluster: "mt1",
        disableStats: true,
        // eslint-disable-next-line no-unused-vars
        authorizer: (channel, options) => {
          return {
            authorize: (socketId, callback) => {
              axios({
                method: "POST",
                url: "http://127.0.0.1:8000/api/broadcasting/auth",
                headers: {
                  Authorization: `Bearer ${token}`,
                },
                data: {
                  socket_id: socketId,
                  channel_name: channel.name,
                },
              })
                .then((response) => {
                  callback(false, response.data);
                })
                .catch((error) => {
                  callback(true, error);
                });
            },
          };
        },
      });

      echo
        .private(`App.Models.User.${data.id}`)
        .listen(".new-message-event", (message) => {
          console.log(message);
        });
    });
  });

export default {};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
