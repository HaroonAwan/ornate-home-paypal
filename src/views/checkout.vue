<template>
  <div>
    <div class="yourProduct">
      <div class="productDeatils">
        <v-card-title>YOUR ORDER</v-card-title>
        <v-data-table
          :headers="headers"
          :items="cartItems"
          hide-default-footer
          class="elevation-1"
        >
          <template v-slot:item.subTotal="{ item }">
            <h4>{{ "$" + item.subTotal }}</h4>
          </template>
        </v-data-table>
        <table id="table" style="width: 100%">
          <tbody>
            <tr>
              <th>Subtotal</th>
              <td>{{ "$" + totalprice }}</td>
            </tr>
            <tr>
              <th>Shipping</th>
              <td>Free Shipping</td>
            </tr>
            <tr>
              <th>Coupon</th>
              <td v-show="clickCpn">
                <v-btn @click="coppen" dark color="green">Apply coupon</v-btn>
              </td>
              <td v-show="showCpn">
                <v-form
                  class="form"
                  @submit.prevent="coupen"
                  ref="form"
                  lazy-validation
                >
                  <v-text-field
                    class="input"
                    type="text"
                    v-model="copen"
                    :rules="[required(), cpn()]"
                    outlined
                  >
                  </v-text-field>
                  <v-btn type="submit" dark color="green">Aplly Coupon</v-btn>
                </v-form>
              </td>
              <td v-show="cpnText">
                <h4>Ornate 10% Discount</h4>
              </td>
            </tr>
            <tr>
              <th>Total</th>
              <td v-if="this.per == true">{{ "$" + totalprice * 0.9 }}</td>
              <td v-else>{{ "$" + totalprice }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="paypal">
        <div class="paypalContent">
          <!-- <p style="margin-top: 50px;">some thing went wrong to payment please try again later</p> -->

          <p>
            Pay via PayPal; you can pay with your credit card if you don’t have
            a PayPal account.
          </p>
          <p>
            Your personal data will be used to process your order, support your
            experience throughout this website, and for other purposes described
            in our privacy policy.
          </p>
        </div>
        <div class="paypalBtn">
          <div ref="paypal"></div>
        </div>
      </div>
    </div>

    <OrderSuccessfull
      v-model="orderSuccesfull"
      :message="this.orderID || 'Null'"
    />
  </div>
</template>

<script>
import { payment } from "../firebase";
import { addDoc } from "@firebase/firestore";

import { required, cpn } from "../utils/validators";
import OrderSuccessfull from "../components/OrderSuccessfull.vue";

export default {
  data() {
    return {
      orderID: "",
      orderSuccesfull: false,
      clickCpn: true,
      showCpn: false,
      cpnText: false,
      per: false,
      copen: null,
      product: {
        price: null,
      },
      carts: [],
      headers: [
        { text: "Product", value: "productTitle" },
        { text: "SubTotal", value: "subTotal" },
      ],
      Order: null,
      payData: null,
    };
  },
  methods: {
    required,
    cpn,
    nullCart() {
      localStorage.setItem("cart", "null");
    },
    loadCarts() {
      this.carts = JSON.parse(localStorage.getItem("cart"));
    },
    refresh() {
      if (this.per == true) {
        this.product.price = this.totalprice * 0.9;
      } else {
        this.product.price = this.totalprice;
      }
    },
    coupen() {
      if (this.$refs.form.validate()) {
        this.per = true;
        this.showCpn = false;
        this.cpnText = true;
      }
    },
    coppen() {
      (this.showCpn = true), (this.clickCpn = false);
    },
    setLoaded: function () {
      window.paypal
        .Buttons({
          createOrder: (data, actions) => {
            return actions.order.create({
              purchase_units: [
                {
                  amount: {
                    value: this.product.price,
                    currency_code: "AUD",
                  },
                },
              ],
            });
          },
          onApprove: async (data, actions) => {
            const order = await actions.order.capture();
            this.Order = order;
            this.orderID = order.id;
            const addedDoc = await addDoc(payment, this.Order);
            console.log(addedDoc);
            this.nullCart();
            this.orderSuccesfull = true;
          },
          onError: (err) => {
            console.log(err);
          },
        })
        .render(this.$refs.paypal);
    },
  },
  computed: {
    cartItems() {
      let cartItem = [];
      if (this.carts != null) {
        cartItem = Object.values(this.carts);
      }
      return cartItem;
      // return Object.values(this.carts);
    },
    totalprice() {
      return this.cartItems
        .reduce((prev, value) => prev + value["subTotal"], 0)
        .toFixed(2);
    },
  },
  mounted() {
    this.loadCarts();
    const script = document.createElement("script");
    script.src =
      "https://www.paypal.com/sdk/js?currency=AUD&client-id=Ad3VfPEIkH_B6ofvpRtGdv0pgK_0dcqvijWBkfidqn17Qnl85QRDuaDAxHLkvZMpaAMHs_f-mIy7GKdE";
    script.addEventListener("load", this.setLoaded);
    document.body.appendChild(script);
    window.setInterval(() => {
      this.refresh();
    }, 300);
  },
  components: { OrderSuccessfull },
};
</script> 

<style scoped>
.form {
  width: 30%;
}

@media (max-width: 600px) {
  .form {
    width: 100%;
  }
}
</style>

<style>
.yourProduct {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.productDeatils {
  width: 95%;
}

.paypal {
  display: flex;
  flex-direction: column;
}

.paypalContent {
  padding: 10px 20px;
}

.paypalBtn {
  margin-left: 20%;
}

@media only screen and (max-width: 600px) {
  .paypalBtn {
    margin: 0px;
  }
}
</style>