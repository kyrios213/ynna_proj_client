<template>
  <v-container>
    <v-row>
      <v-col align="center"><h1>Register</h1></v-col>
    </v-row>
    <v-row v-if="error">
      <v-col align="center"
        ><v-alert border="left" prominent shaped type="error">{{
          error
        }}</v-alert></v-col
      >
    </v-row>
    <v-row>
      <v-col offset-md="2" md="8" align="center"
        ><validation-observer ref="observer" v-slot="{ invalid }">
          <form @submit.prevent="submit">
            <validation-provider
              v-slot="{ errors }"
              name="Username"
              rules="required|min:5"
            >
              <v-text-field
                v-model="username"
                :error-messages="errors"
                label="Username"
                required
              ></v-text-field>
            </validation-provider>

            <validation-provider
              v-slot="{ errors }"
              name="Password"
              rules="required|min:6"
            >
              <v-text-field
                v-model="password"
                :error-messages="errors"
                label="Password"
                type="password"
                required
              ></v-text-field>
            </validation-provider>

            <v-btn class="mr-4" type="submit" :disabled="invalid">
              submit
            </v-btn>
            <v-btn @click="clear"> clear </v-btn>
          </form>
        </validation-observer></v-col
      >
    </v-row>
  </v-container>
</template>

<script>
import { required, min } from "vee-validate/dist/rules";
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

extend("min", {
  ...min,
  message: "{_field_} may not be shorter than {length} characters",
});

export default {
  components: {
    ValidationProvider,
    ValidationObserver,
  },
  data: () => ({
    username: "",
    password: "",
    error: "",
  }),
  mounted() {
    if (this.$cookies.get("token")) {
      this.$router.push({ name: "Home" });
    }
  },
  methods: {
    async submit() {
      this.$refs.observer.validate();
      const res = await fetch(
        "https://christmas-chooser-api.herokuapp.com/api/user/signup/",
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            username: this.username,
            password: this.password,
          }),
        }
      );
      const data = await res.json();
      if (data.error) {
        this.error = data.error;
        return;
      }
      await this.$router.push({ name: "Login" });
    },
    clear() {
      this.username = "";
      this.password = "";
      this.error = "";
      this.$refs.observer.reset();
    },
  },
};
</script>

<style></style>
