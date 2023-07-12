<template>
  <div class="main">
    <form id="payment-form" @submit.prevent="handleSubmit">
      <div class="content">
        <div id="card-element"></div>
        <button id="submit">add card</button>
      </div>
    </form>
    <div class="listcard">
      <button @click="handleListCard">list card</button>
      <div class="note">
        add card sẽ là card thứ 2 ( card đầu là card default )
      </div>
      <div class="cards">
        <div class="info" v-for="(item, index) in listCard" :key="index">
          cardId: {{ item.id }} , brand: {{ item.brand }} , last4 :
          {{ item.last4 }} , customer: {{ item.customer }} , exp_month:
          {{ item.exp_month }} , exp_year: {{ item.exp_year }} ,
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import { loadStripe } from "@stripe/stripe-js";
import Stripe from "stripe";

const stripeJs = new Stripe(
  "sk_test_51LFV4fAvLtM7cAjKd7Z7XJHcuS6qvz55a9xyjAtj9GBLtQKoQeIY6JRFn85LVGBgOVnDtjQ56TpTzIq47cbRynqH00Ubs1iXDA"
);
let listCard = ref([]);
let stripePromise;
let cardElement;
const customerId = "cus_MNclDrTbC1tFon";

onMounted(async () => {
  stripePromise = await loadStripe(
    "pk_test_51LFV4fAvLtM7cAjK4GAALyctkfSVmJ0YrPSVuU2mJA6h0O2ZgP0w1fdW6TSgahnv1WZxkbA7sE6RSEBv024xgenj00o8szPE3E"
  );
  const elements = stripePromise.elements();
  cardElement = elements.create("card", {
    iconStyle: "solid",
    hidePostalCode: true,
    style: {
      base: {
        fontSize: "16px",
        color: "#424770",
        "::placeholder": {
          color: "#aab7c4",
        },
      },
      invalid: {
        color: "#9e2146",
      },
    },
  });
  cardElement.mount("#card-element");
});

const handleSubmit = async () => {
  try {
    const { token } = await stripePromise.createToken(cardElement);
    await stripeJs.customers.createSource(
      customerId, // customerId
      { source: token.id }
    );
    alert("success");
  } catch (error) {
    alert("error");
  }
};
const handleListCard = async () => {
  const list = await stripeJs.customers.listSources(customerId, {
    object: "card",
  });
  listCard.value = list.data;
};
</script>

<style scoped>
.main {
  display: flex;
  justify-content: center;
  flex-wrap: nowrap;
  flex-direction: column;
}
#payment-form {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}
#payment-form .content {
  width: 400px;
}
</style>
