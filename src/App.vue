<template>
  <v-app id="inspire">
    <v-app-bar app color="white" flat>
      <v-container class="py-0 fill-height d-flex">
        <v-btn icon color="grey" :to="{ name: 'Home' }">
          <v-icon>mdi-home</v-icon>
        </v-btn>
        <div v-if="user">
          <v-btn class="mx-2" color="error" @click="logout"> Logout</v-btn>
        </div>
        <div v-else>
          <v-btn class="mx-2" color="primary" :to="{ name: 'Login' }">
            Login
          </v-btn>
          <v-btn class="mx-2" color="success" :to="{ name: 'Register' }">
            Register
          </v-btn>
        </div>
      </v-container>
    </v-app-bar>

    <v-main class="grey lighten-3">
      <v-container>
        <v-row>
          <v-col cols="2">
            <v-sheet rounded="lg">
              <v-list color="transparent" v-if="user">
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title> User: </v-list-item-title>
                    <v-list-item-subtitle>{{
                      user.username
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title> Gifting: </v-list-item-title>
                    <v-list-item-subtitle>{{
                      user.drawnUser
                    }}</v-list-item-subtitle>
                  </v-list-item-content>
                </v-list-item>
                <v-list-item v-if="user.room">
                  <v-list-item-content>
                    <v-list-item-title> Room: </v-list-item-title>
                    <v-list-item-subtitle>{{ user.room }}</v-list-item-subtitle>
                    <v-list-item-subtitle
                      ><v-btn
                        small
                        :to="{ name: 'Room', params: { id: user.room } }"
                        >Join</v-btn
                      ></v-list-item-subtitle
                    >
                    <v-list-item-subtitle class="mt-2"
                      ><v-btn small @click="leaveRoom" color="error"
                        >Leave</v-btn
                      ></v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-sheet>
          </v-col>

          <v-col>
            <v-sheet min-height="70vh" rounded="lg">
              <router-view @login="getUser" @update="getUser" />
            </v-sheet>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      user: null,
    };
  },
  created() {
    const token = this.$cookies.get("token");
    if (token) this.getUser();
  },
  methods: {
    async getUser() {
      const token = this.$cookies.get("token");
      const res = await fetch(
        "https://christmas-chooser-api.herokuapp.com/api/user/user",
        {
          method: "GET",
          headers: {
            "x-auth-token": token,
          },
        }
      );
      const data = await res.json();
      this.user = {
        username: data.username,
        drawnUser: data.drawnUser,
        room: data.room,
      };
    },
    async leaveRoom() {
      const token = this.$cookies.get("token");
      const res = await fetch(
        `https://christmas-chooser-api.herokuapp.com/api/room/leave-room/${this.user.room}`,
        {
          method: "POST",
          headers: {
            "x-auth-token": token,
          },
        }
      );
      const data = await res.json();
      this.error = data.error;
      await this.$router.push({ name: "Home" });
      await this.getUser();
    },
    logout() {
      this.$cookies.remove("token");
      this.user = null;
      this.$router.push({ name: "Home" });
    },
  },
};
</script>
