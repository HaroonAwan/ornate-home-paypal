<template>
  <v-dialog :value="flag" @input="$emit('value', false)" width="470">
    <v-card style="padding: 10px 0">
      <sweetalert-icon icon="success" />

      <h4 id="title">Your Order is confirmed</h4>

      <h5 id="text">Thank you for ordering from Ornate Home!</h5>

      <h6 style="padding: 5px 20px">
        We're excited for you to receive your order
        <span style="background-color: yellow; padding: 0px 6px">
          ID:
          <span style="color: green; font-weight: bold">{{
            message
          }}</span></span
        >
        and will notify you once it's on its way. We hope you had a great
        shopping experience!
      </h6>
      <v-card-actions>
        <v-spacer />
        <v-btn color="#38872c" dark elevation="0" @click="trackOrder"
          >Track Order</v-btn
        >
        <v-btn color="#38872c" dark elevation="0" @click="$emit('value', false)"
          >Ok</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  name: "ErrorDialog",

  methods: {
    trackOrder() {
      this.$router.push("/trackOrders");
    },
  },

  model: {
    prop: "flag",
    event: "value",
  },

  props: {
    flag: {
      type: Boolean,
      default: false,
    },

    message: {
      type: String,
      default: "",
    },
  },
};
</script>

<style scoped>
#title {
  color: green;
  font-family: google-sans, sans-serif;
  font-size: 18px;
  /* font-weight: 600; */
  display: flex;
  justify-content: center;
}

#text {
  display: flex;
  justify-content: center;
  padding: 5px 20px;
}

@media only screen and (max-width: 600px) {
  #title {
    font-size: 15px;
    padding-bottom: 10px;
  }
}
</style>
