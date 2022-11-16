<template>
  <div>
    <div class="aboutHeader">
      <h2>Contact Us</h2>
      <div class="aboutHeader-link">
        <p>
          <router-link style="text-decoration: none" to="/">
            <v-icon style="margin-top: -4px"> mdi-home </v-icon>
          </router-link>
          <router-link
            style="text-decoration: none; color: black"
            to="/contactUs"
          >
            > Contact Us
          </router-link>
        </p>
      </div>
    </div>

    <div>
      <v-card class="mx-auto my-12 pa-10" max-width="1000">
        <div
          class="headerTitle"
          style="margin-top: -10px; padding-bottom: 25px"
        >
          <h2>
            Contact <span style="color: #38872c; font-weight: 700">Us</span>
          </h2>

          <div class="hr">
            <hr style="border: 2px solid green; width: 3px" />
            <hr style="border: 2px solid green; width: 3px" />
            <hr style="border: 2px solid green; width: 3px" />
            <hr style="border: 2px solid green; width: 100px" />
          </div>
        </div>

        <v-form lazy-validation ref="form" @submit.prevent="submit">
          <v-col>
            <v-text-field
              :rules="[required()]"
              label="Your Name"
              v-model="yourName"
              required
              outlined
            >
            </v-text-field>
          </v-col>

          <v-col>
            <v-text-field
              :rules="[required(), email()]"
              label="Your Email"
              v-model="yourEmail"
              required
              outlined
            >
            </v-text-field>
          </v-col>

          <v-col>
            <v-text-field
              :rules="[required()]"
              label="Subject"
              v-model="subject"
              required
              outlined
            >
            </v-text-field>
          </v-col>

          <v-col>
            <v-textarea
              label="Your Message (Optional)"
              v-model="message"
              required
              outlined
            >
            </v-textarea>
          </v-col>

          <v-col>
            <p v-show="success" style="color: green">
              Message send Succesfully!
            </p>
            <p v-show="err" style="color: red">
              Something went wrong! Please try again later
            </p>
          </v-col>

          <v-col>
            <v-btn dark style="background-color: #38872c" type="submit"
              >Submit</v-btn
            >
          </v-col>
        </v-form>
      </v-card>
    </div>

    <LoadingDialog v-model="loader" message="Sending" />
  </div>
</template>

<script>
import { required, email } from "@/utils/validators";
import LoadingDialog from "../components/LoadingDialog.vue";

export default {
  data() {
    return {
      yourName: null,
      yourEmail: null,
      subject: null,
      message: null,
      err: false,
      success: false,
      loader: false,
    };
  },
  methods: {
    email,
    required,

    async submit() {
      if (this.$refs.form.validate()) {
        this.loader = true;
        await fetch("https://formsubmit.co/ajax/ornatehome1@gmail.com", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
            Accept: "application/json",
          },
          body: JSON.stringify({
            name: this.yourName,
            email: this.yourEmail,
            subject: this.subject,
            message: this.message,
          }),
        }).then((res) => {
          if (res.status === 200) {
            this.loader = false;
            this.success = true;
            this.err = false;
            this.$refs.form.reset();
          } else {
            this.loader = false;
            this.success = false;
            this.err = true;
            this.alert = "Something went wrong! Please try again later";
          }
        });
      }
    },
  },
  components: {
    LoadingDialog,
  },
};
</script>