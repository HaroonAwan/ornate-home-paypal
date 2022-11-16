<template>
  <div>
    <div v-show="trackForm">
      <div class="aboutHeader">
        <h2>Track Orders</h2>
        <div class="aboutHeader-link">
          <p>
            <router-link style="text-decoration: none" to="/">
              <v-icon style="margin-top: -4px"> mdi-home </v-icon>
            </router-link>
            <router-link
              style="text-decoration: none; color: black"
              to="/trackOrders"
            >
              > Track Orders
            </router-link>
          </p>
        </div>
      </div>

      <p class="tOrder">
        To track your order please enter your Order ID in the box below and
        press the "Track" button. This was given to you on your receipt and in
        the confirmation email you should have received.
      </p>

      <div>
        <v-card id="trackForm" class="mx-auto pa-10" style="width: 80%">
          <div
            class="headerTitle"
            style="margin-top: -10px; padding-bottom: 25px"
          >
            <h2>
              Track <span style="color: #38872c; font-weight: 700">Order</span>
            </h2>

            <div class="hr">
              <hr style="border: 2px solid green; width: 3px" />
              <hr style="border: 2px solid green; width: 3px" />
              <hr style="border: 2px solid green; width: 3px" />
              <hr style="border: 2px solid green; width: 100px" />
            </div>
          </div>

          <v-form lazy-validation ref="form" @submit.prevent="loadData">
            <v-col>
              <v-text-field
                :rules="[required()]"
                label="Order Id"
                v-model="orderId"
                required
                outlined
              >
              </v-text-field>
            </v-col>

            <!-- <v-col>
              <v-text-field
                :rules="[required(), email()]"
                label="Billing Email"
                v-model="orderEmail"
                required
                outlined
              >
              </v-text-field>
            </v-col> -->

            <v-col>
              <v-btn dark style="background-color: #38872c" type="submit"
                >Track</v-btn
              >
            </v-col>
          </v-form>
        </v-card>
      </div>
    </div>

    <div v-show="trackDetails" style="margin-top: 50px">
      <v-card
        class="mx-auto pa-10"
        id="orderDetails"
        style="
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: center;
          width: 80%;
        "
      >
        <div
          class="headerTitle"
          style="margin-top: -10px; padding-bottom: 25px"
        >
          <h2>
            Order <span style="color: #38872c; font-weight: 500">Details</span>
          </h2>

          <div class="hr">
            <hr style="border: 2px solid green; width: 3px" />
            <hr style="border: 2px solid green; width: 3px" />
            <hr style="border: 2px solid green; width: 3px" />
            <hr style="border: 2px solid green; width: 100px" />
          </div>
        </div>
        <v-col style="display: flex; justify-content: space-between">
          <h4>ID:</h4>
          <p>{{ this.id }}</p>
        </v-col>

        <v-col style="display: flex; justify-content: space-between">
          <h4>Name:</h4>
          <p>{{ this.name }}</p>
        </v-col>

        <v-col style="display: flex; justify-content: space-between">
          <h4>Email:</h4>
          <p>{{ this.Email }}</p>
        </v-col>

        <v-col style="display: flex; justify-content: space-between">
          <h4>Payment:</h4>
          <p>{{ this.amount }}</p>
        </v-col>

        <v-col style="display: flex; justify-content: space-between">
          <h4>Address:</h4>
          <p>{{ this.address }}</p>
        </v-col>
        <v-col>
          <h4 style="background-color: #ddd; padding: 5px 10px">
            Your order is on the way you can get more information at
            <strong style="background-color: yellow; padding: 0px 5px"
              >ornatehome1@gmail.com</strong
            >
          </h4>
        </v-col>
      </v-card>
    </div>

    <LoadingDialog v-model="loader" message="Please wait" />
  </div>
</template>

<script>
import { required, email } from "@/utils/validators";
import { getDocs, query, where } from "@firebase/firestore";
import { payment } from "../firebase";
import LoadingDialog from "../components/LoadingDialog.vue";

export default {
  data() {
    return {
      loader: false,
      orderId: null,
      trackForm: true,
      trackDetails: false,
      id: null,
      name: null,
      Email: null,
      amount: null,
      address: null,
    };
  },
  methods: {
    email,
    required,

    async loadData() {
      this.loader = true;
      try {
        if (this.$refs.form.validate()) {
          let q;
          q = query(payment, where("id", "==", this.orderId));
          const snapshot = await getDocs(q);
          const data = [];
          snapshot.docs.map((e) => {
            data.push({ id: e.id, ...e.data() });
          });
          if (data.length >= 1) {
            this.trackForm = false;
            this.trackDetails = true;
            this.id = data[0].id;
            this.name = data[0].purchase_units[0].shipping.name.full_name;
            this.Email = data[0].payer.email_address;
            this.amount =
              data[0].purchase_units[0].payments.captures[0].amount.value;
            this.address =
              data[0].purchase_units[0].shipping.address.address_line_1;
            this.loader = false;
          } else {
            this.loader = false;
            alert("Record not found");
          }
        }
      } catch (e) {
        console.log(e);
      }
    },
  },
  components: {
    LoadingDialog,
  },
};
</script>

<style>
.tOrder {
  padding: 50px;
}

@media only screen and (max-width: 450px) {
  .tOrder {
    padding: 20px;
  }

  #orderDetails,
  #trackForm {
    width: 95% !important;
    padding: 0px !important;
  }
}
</style>