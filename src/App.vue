<template>
  <div id="app">
    <header>
      <h1>{{ sitename }}</h1>
      <button @click="showCheckout">{{ cart.length }} Checkout</button>
    </header>
    <main>
      <component :is="currentView"></component>
    </main>
  </div>
</template>

<script>
import ProductList from "./components/ProductList.vue";
import Checkout from "./components/Checkout.vue";

export default {
  name: "App",
  data() {
    return {
      sitename: "Lecture app",
      cart: [], 
      currentView: ProductList,
    };
  },
  methods: {
    showCheckout() {
      if (this.currentView === ProductList) {
          this.currentView = Checkout
      } else {
        this.currentView = ProductList
      }
    },
  },
};
</script>
