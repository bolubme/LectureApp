<template>
  <div id="app">
    <!-- Loader section -->
    <div v-if="isDataLoading" class="loader-container">
      <div class="loader"></div>
    </div>
    <!-- ======= NavBar Section ======= -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <div class="container-fluid justify-content-between">
        <div class="d-flex">
          <a class="navbar-brand me-2 mb-1 d-flex align-items-center"
            >LECTURE</a
          >
        </div>

        <ul class="navbar-nav flex-row d-md-flex">
        </ul>

        <ul class="navbar-nav flex-row">
          <a class="text-reset me-3" href="#">
            <button @click="showCheckout" class="checkout" :disabled="cart.length === 0">
              <i class="fas fa-shopping-cart"></i>
              <span class="cartNumber">{{ cartItemCount }}</span>
            </button>
          </a>
        </ul>
      </div>
    </nav>
    <!-- ======= NavBar Section Ends ======= -->

    <section id="products" class="products">
      <main>
        <component
          :is="currentView"
          :filteredProducts="filteredProducts"
          :products="products"
          :cart="cart" 
          @add-to-cart="addToCart"
          @remove-product="removeProduct"
          @order="orderNow"
          @sort-field-change="handleSortFieldChange"
          @search-change="handleSearch"
          @search="search"
        ></component>
      </main>
    </section>
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
      products: [],
      cart: [],
      showProducts: true,
      currentView: ProductList,
      searchWord: "",
      searchTemp: [],
      isLoading: true,
      orderDetails: null,
      filterOption: "",
      sortField: "",
      
    };
  },
  methods: {
    init() {
      this.currentView = ProductList;
      this.cart = [];
      this.loadLessonData();
    },

    updateAvailability(id, spaceNum) {
      fetch("https://lecture-app.onrender.com/collections/lectures/" + id, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          availableInventory: spaceNum,
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          return data;
        })
        .catch((err) => {
          return data;
        });
    },

    loadLessonData: function () {
      console.log("Collecting lessons from database...");

      const url = "https://lecture-app.onrender.com/collections/lectures";

      fetch(url)
        .then((response) => {
          if (!response.ok) {
            throw new Error(`Failed to fetch data. Status: ${response.status}`);
          }
          return response.json();
        })
        .then((json) => {
          this.products = json;
          this.isLoading = false;
        })
        .catch((error) => {
          console.error("Error loading lesson data:", error);
          this.isLoading = false;
        });
    },

    productIndex: function (product) {
      return this.products.findIndex((p) => p.id === product.id);
    },

    canAddToCart: function (product) {
      const productIndex = this.productIndex(product);
      return (
        productIndex !== -1 &&
        this.products[productIndex].availableInventory > 0
      );
    },
    showCheckout: function() {
      if (this.currentView === Checkout) {
        this.currentView = ProductList;
      } else {
          this.currentView = Checkout;
      }
    },

    addToCart: function(product) {
      const foundProduct = this.searchWord === "" ? this.products.find((p) => p.id === product.id) : this.searchTemp.find((p) => p.id === product.id);

      console.log(foundProduct)    
    
      if (foundProduct && this.canAddToCart(foundProduct)) {
        this.cart.push(foundProduct);
        foundProduct.availableInventory--;
      }
    },

    removeProduct: function(product) {
      if (this.cart.length === 1) {
        this.currentView = ProductList;
      }
      const index = this.cart.indexOf(product);
      if (index !== 1) {
        this.cart.splice(index, 1);
        const productIndex = this.productIndex(product);
        this.products[productIndex].availableInventory++;
      }
    },

    orderNow(orderDetails) {
      const orderData = [];
      this.cart.forEach((lectures) => {
        orderData.push({ id: lectures.id, courseName: lectures.subject });
        this.updateAvailability(lectures._id, lectures.availableInventory);
      });

      alert("Order Submitted!");
      const url = "https://lecture-app.onrender.com/collections/orders";
      const data = {
        name: orderDetails.name,
        phone: orderDetails.phone,
        lessonsOrdered: orderData,
      };
      this.init();

      fetch(url, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(data),
      })
        .then((response) => response.json())
        .then((data) => console.log("Success:", data))
        .catch((error) => console.error("Error:", error));
    },

    handleSortFieldChange(sort){
      this.filterOption = sort.filterOption
      this.sortField = sort.sortField
    },

    handleSearch(searchText){
      this.searchWord = searchText 
    },

    findProductsByObjects(searchObjects) {
      const result = [];

      searchObjects.forEach((searchObj) => {
        const foundProduct = this.products.find((product) => {
          return product._id === searchObj._id;
        });

        if (foundProduct) {
          result.push(foundProduct);
        }
      });

      return result;
    },

    search() {
      if (this.searchWord.length > 0) {
        this.fetchSearchResults();
      } else {
      }
    },

    fetchSearchResults() {
      const query = encodeURIComponent(this.searchWord);
      const url = `https://lecture-app.onrender.com/collections/lectures/search/${query}`

      fetch(url)
        .then((response) => {
          if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
          }
          return response.json();
        })
        .then((data) => {
          // this.products = data
          this.searchTemp = this.findProductsByObjects(data);
        })
        .catch((error) => {
          console.error("Error fetching search results:", error);
        });
    },


  },

  computed: {
    isDataLoading() {
      return this.isLoading;
    },

    cartItemCount: function () {
      return this.cart.length || "";
    },

    filteredProducts() {
      const field = this.sortField.toLowerCase();
      let order = this.filterOption === "ascending" ? 1 : -1;
    
      let filterList;
      if (this.searchWord === "") {
        filterList = this.products;
      } else {
        filterList = this.searchTemp;
      }
    
      return filterList.slice().sort((a, b) => {
        if (field === "location")
          return order * a.location.localeCompare(b.location);
        if (field === "price") return order * (a.price - b.price);
        if (field === "subject")
          return order * a.subject.localeCompare(b.subject);
        if (field === "spaces")
          return order * (a.availableInventory - b.availableInventory);
        if (field === "rating") return order * (a.rating - b.rating);
        return 0;
      });
    },
  },


  created: function(){
    this.loadLessonData();
    if ("serviceWorker" in navigator) {
      navigator.serviceWorker.register("service-worker.js");
    }
  }
};
</script>

<style>
/* Add these styles for the loader */
.loader-container {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 150vh;
  z-index: 1000;
}

.loader {
  border: 8px solid #f3f3f3;
  border-top: 8px solid #3498db;
  border-radius: 50%;
  width: 50px;
  height: 50px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.checkout {
  border: none;
  background-color: white;
  color: black;
  padding: 7px;
  border-radius: 15px;
}

.checkout:hover {
  background-color: #fab300;
}

.cartNumber {
  color: black;
  text-align: center;
  font-size: 20px;
  font-weight: bolder;
  border-radius: 50%;
}

.fa-shopping-cart {
  color: black;
  font-size: 30px;
}
</style>
