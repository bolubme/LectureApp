<template>
  <!-- Checkout page  -->
  <div>
    <div class="checkout-Body">
      <div class="container">
        <h2 class="checkout-header">Checkout</h2>

        <div class="row">
          <div v-for="productId in cart" class="col-lg-4 col-md-6 d-flex">
            <div class="lecture">
              <img
                :src="productId.image"
                :alt="productId.subject"
                class="img-fluid"
              />
              <div class="lecture-details">
                <h2 v-text="productId.subject"></h2>
                <h6>
                  Location:
                  <span v-text="productId.location"></span>
                </h6>
                <h6>
                  Price:
                  <span v-text="productId.price"></span>
                </h6>
                <div>
                  <span v-for="n in productId.rating">★</span>
                  <span v-for="n in 5 - productId.rating">☆</span>
                </div>
              </div>
              <button class="btn" @click="removeProduct(productId)">
                Remove
              </button>
            </div>
          </div>
        </div>

        <div style="margin-top: 50px" class="checkout-form">
          <form>
            <p>
              <label for="name">Name</label>
              <input
                id="name"
                v-model.trim="order.firstName"
                required
                @input="validateName"
              />
              <span style="color: red; font-size: 12px" v-if="!isNameValid"
                >Input Only characters</span
              >

              <label for="number">Phone Number</label>
              <input
                id="number"
                v-model.trim="order.phoneNumber"
                required
                @input="validatePhoneNumber"
              />
              <span style="color: red; font-size: 12px" v-if="!isPhoneValid"
                >Number only</span
              >
            </p>
            <button
              style="border: 1px solid black"
              class="btn"
              v-on:click="orderNow"
              v-if="isFormValid"
            >
              Place Order
            </button>
            <button
              style="border: 1px solid black"
              class="btn"
              disabled="disabled"
              v-else
            >
              Place Order
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  nmae: "Checkout",
  data() {
    return {
      order: {
        firstName: "",
        phoneNumber: "",
      },
      isNameValid: false,
      isPhoneValid: false,
    };
  },
  props: ["filteredProducts", "cart"],
  methods: {
    removeProduct: function (product) {
      this.$emit("remove-product", product);
    },

    validateName() {
      const pattern = /^[A-Za-z\s]+$/;
      this.isNameValid = pattern.test(this.order.firstName);
    },
    validatePhoneNumber() {
      const pattern = /^(\(?(0|\+44)[1-9]{1}\d{1,4}?\)?\s?\d{3,4}\s?\d{3,4})$/;
      this.isPhoneValid = pattern.test(this.order.phoneNumber);
    },

    orderNow() {
      this.$emit("order", {
        name: this.order.firstName,
        phone: this.order.phoneNumber,
      });
    },
  },
  computed: {
    isFormValid() {
      return this.isNameValid && this.isPhoneValid;
    },
  },
};
</script>

<style></style>
