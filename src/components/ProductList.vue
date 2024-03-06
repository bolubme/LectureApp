<template>
  <div>
    <div class="container">
      <!-- ======= Sorting section ======= -->
      <div class="row mb-3">
        <div class="col-md-3">
          <label for="sortSelect">Sort By:</label>
          <select id="sortSelect" class="form-control" v-model="sortField">
            <option value="Location">Location</option>
            <option value="Price">Price</option>
            <option value="Subject">Subject</option>
            <option value="Spaces">Spaces</option>
            <option value="Rating">Rating</option>
          </select>
        </div>
        <div class="col-md-3">
          <label>Filter By:</label>
          <div class="form-check">
            <input
              type="radio"
              class="form-check-input"
              name="filterOption"
              id="filterAscending"
              value="ascending"
              v-model="filterOption"
            />
            <label class="form-check-label" for="filterAscending"
              >Ascending</label
            >
          </div>
          <div class="form-check">
            <input
              type="radio"
              class="form-check-input"
              name="filterOption"
              id="filterDescending"
              value="descending"
              v-model="filterOption" 
            />
            <label class="form-check-label" for="filterDescending"
              >Descending</label
            >
          </div>
        </div>
        <form class="input-group w-auto my-auto d-sm-flex">
            <input
              autocomplete="off"
              type="search"
              class="form-control rounded"
              placeholder="Search"
              style="min-width: 125px"
              v-model="searchWord"
              @input="search" 
              
            />
          </form>
      </div>
      <!-- ======= Sorting section Ends ======= -->
      <!-- ======= Product Cards ======= -->
      <div class="row">
        <div
          v-for="product in filteredProducts"
          class="col-lg-4 col-md-6 d-flex"
        >
          <div class="lecture">
            <img
              :src="product.image"
              :alt="product.subject"
              class="img-fluid"
            />
            <div class="lecture-details">
              <h2 v-text="product.subject"></h2>
              <h6>Location: <span v-text="product.location"></span></h6>
              <h6>Price: <span v-text="product.price"></span></h6>
              <div>
                <span v-for="n in product.rating">★</span>
                <span v-for="n in 5 - product.rating">☆</span>
              </div>
              <div class="">
                <span style="color: red" v-if="product.availableInventory === 0"
                  >All out!</span
                >
                <span v-else-if="product.availableInventory < 5">
                  Only {{ product.availableInventory }} left!
                </span>
                <span v-else>Buy now!</span>
              </div>
            </div>

            <button
              class="btn"
              v-on:click="addToCart(product)"
              v-if="canAddToCart(product)"
            >
              Add to Cart
            </button>
            <button class="btn" disabled="disabled" v-else>Add to Cart</button>
          </div>
        </div>
      </div>
      <!-- ======= Product Cards Ends======= -->
    </div>
  </div>
</template>

<script>
export default {
  name: "ProductList",
  props: ["filteredProducts", "cart", "products"],
  data() {
    return {
        filterOption: "ascending",
        sortField: "Location",
        searchWord: ""
    }
  },

  methods: {
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

    addToCart: function (product) {
      this.$emit("add-to-cart", product);
    },

    onSortFieldChange() {
        this.$emit("sort-field-change", {
            sortField: this.sortField,
            filterOption: this.filterOption,
        });
    },

    searchChange() {
      this.$emit("search-change", this.searchWord);
    },

    search(){
        this.$emit("search")
    }
  },

  watch: {
    // Watch for changes in sortField and filterOption and emit event
    sortField: function(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.onSortFieldChange();
      }
    },
    filterOption: function(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.onSortFieldChange();
      }
    },
    searchWord: function(newVal, oldVal) {
      if (newVal !== oldVal) {
        this.searchChange();
      }
    }

  }
  
};
</script>

<style>
.lecture {
  text-align: left;
  margin-bottom: 20px;
  background: #fff;
  border: 1px solid #eef0ef;
}

.lecture img {
  margin: -1px -1px 30px -1px;
}

.lecture .lecture-details {
  padding: 0 20px 30px 20px;
}

.lecture h2 {
  font-weight: 700;
  margin-bottom: 2px;
  font-size: 34px;
}

.lecture .btn {
  width: 100%;
  border: 1px solid black;
}

.lecture .btn:hover {
  background-color: #fab300;
}
</style>
