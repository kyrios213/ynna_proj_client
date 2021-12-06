<template>
  <v-container>
    <v-row>
      <v-col align="center"><h1>Home</h1></v-col>
    </v-row>
    <v-row v-if="error" class="mb-10">
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
    <v-row class="mt-10">
      <v-col offset-md="2" md="8" align="center"
        ><validation-observer ref="observer" v-slot="{ invalid }">
          <form @submit.prevent="joinRoom">
            <validation-provider
              v-slot="{ errors }"
              name="Room"
              rules="required"
            >
              <v-text-field
                v-model="room"
                :error-messages="errors"
                label="Room"
                required
              ></v-text-field>
            </validation-provider>

            <v-btn
              class="mr-4"
              :disabled="invalid"
              color="error"
              @click="joinRoom"
            >
              Join Room
            </v-btn>
            <v-btn @click="createRoom" color="primary"> Create Room </v-btn>
          </form>
        </validation-observer></v-col
      >
    </v-row>
  </v-container>
</template>

<script>
import { required } from "vee-validate/dist/rules";
import {
  extend,
  ValidationObserver,
  ValidationProvider,
  setInteractionMode,
} from "vee-validate";

setInteractionMode("eager");

extend("required", {
  ...required,
  message: "{_field_} can not be empty",
});

export default {
  components: {
    ValidationProvider,
    ValidationObserver,
  },
  data: () => ({
    room: "",
    error: "",
  }),
  methods: {
    async joinRoom() {
      // this.$refs.observer.validate();
      const token = this.$cookies.get("token");
      const res = await fetch(
        `https://christmas-chooser-api.herokuapp.com/api/room/join-room/${this.room}`,
        {
          method: "POST",
          headers: {
            "x-auth-token": token,
          },
        }
      );
      const data = await res.json();
      if (data.error) {
        this.error = data.error;
        return;
      }
      await this.$emit("update");
      await this.$router.push({ name: "Room", params: { id: this.room } });
    },
    async createRoom() {
      const token = this.$cookies.get("token");
      const res = await fetch(
        `https://christmas-chooser-api.herokuapp.com/api/room/create-room`,
        {
          method: "POST",
          headers: {
            "x-auth-token": token,
          },
        }
      );
      const data = await res.json();
      if (data.error) {
        this.error = data.error;
        return;
      }
      this.$emit("update");
      this.$router.push({ name: "Room", params: { id: data.roomId } });
    },
  },
};
</script>

<style></style>
