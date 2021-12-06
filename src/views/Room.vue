<template>
  <v-container>
    <v-row class="my-10">
      <v-col align="center">
        <h1>Room</h1>
        <br />
        <p>Room id: {{ id }}</p>
      </v-col>
    </v-row>
    <v-row v-if="error">
      <v-col align="center"
        ><v-alert
          border="left"
          prominent
          shaped
          type="error"
          @click="error = ''"
          >{{ error }}</v-alert
        ></v-col
      >
    </v-row>
    <v-row v-if="success">
      <v-col align="center"
        ><v-alert
          border="left"
          prominent
          shaped
          type="success"
          @click="success = null"
          >You are now gifting user: {{ success.user }}</v-alert
        ></v-col
      >
    </v-row>

    <v-row>
      <v-col md="4" align="left">
        <v-card class="mx-auto" max-width="300" tile>
          <v-list dense>
            <v-subheader>Users</v-subheader>
            <v-list-item-group color="primary">
              <v-list-item v-for="user in users" :key="user">
                <v-list-item-content>
                  <v-list-item-title v-text="user"></v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </v-list-item-group>
          </v-list>
        </v-card>
      </v-col>

      <v-col md="8" align="center">
        <v-row class="my-10"></v-row>
        <v-row>
          <v-col
            ><v-btn color="secondary" elevation="20" x-large @click="drawUser"
              >Draw</v-btn
            ></v-col
          >
        </v-row>
        <v-row></v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      room: null,
      error: "",
      users: [],
      success: null,
    };
  },
  props: ["id"],
  async mounted() {
    this.getRoom();
  },
  methods: {
    async getUsers() {
      for (let i = 0; i < this.room.users.length; i++) {
        const res = await fetch(
          `https://christmas-chooser-api.herokuapp.com/api/user/get-user/${this.room.users[i]}`
        );
        const data = await res.json();
        this.users.push(data);
      }
    },
    async getRoom() {
      const token = this.$cookies.get("token");
      try {
        const res = await fetch(
          `https://christmas-chooser-api.herokuapp.com/api/room/get-room/${this.id}`,
          {
            method: "GET",
            headers: {
              "x-auth-token": token,
            },
          }
        );
        const data = await res.json();
        if (data.error) {
          return (this.error = data.error);
        }
        this.room = data;
        await this.getUsers();
      } catch (err) {
        console.log(err);
      }
    },
    async drawUser() {
      this.error = "";
      const token = this.$cookies.get("token");
      const random = Math.floor(Math.random() * this.room.users.length);
      const res = await fetch(
        `https://christmas-chooser-api.herokuapp.com/api/room/giftTo/${this.room.users[random]}`,
        {
          method: "PATCH",
          headers: {
            "x-auth-token": token,
          },
        }
      );
      const data = await res.json();
      if (data.error) {
        return (this.error = data.error);
      }
      this.success = data;
      await this.$emit("update");
    },
  },
};
</script>

<style></style>
