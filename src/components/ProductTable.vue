<template>
  <div class="card">
    <h2>Product Table</h2>
    <div class="container">
      <div class="search-box">
        <button class="btn add" @click="showDialog()">Add Product</button>
        <input
          type="text"
          v-model="search"
          placeholder="Search by product title..."
        />
      </div>
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Image</th>
            <th>Title</th>
            <th>Price</th>
            <th>Category</th>
            <th>Description</th>
            <th>Update</th>
            <th>Delete</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product, index) in filteredProducts" :key="product.id">
            <td>{{ index + 1 }}</td>
            <td>
              <img
                :src="product.image"
                height="30"
                width="30"
                alt="product Image"
              />
            </td>
            <td>{{ product.title }}</td>
            <td>${{ product.price }}</td>
            <td>{{ product.category }}</td>
            <td>{{ product.description }}</td>
            <td>
              <button class="btn update" @click="onUpdate(product)">
                Update
              </button>
            </td>
            <td>
              <button class="btn delete" @click="onDelete(product)">
                Delete
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <AddUpdateProduct
      ref="addUpdateProduct"
      @add="getProductsDetails"
      @update="updateProduct"
    />
    <DeleteProduct ref="deleteProduct" @delete="onDeleteProduct($event)" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, watch } from "vue";
import { IProductDetails } from "./src/product.ts";
import AddUpdateProduct from "./AddUpdateProduct.vue";
import DeleteProduct from "./DeleteProduct.vue";
import axios from "axios";

const products = ref<IProductDetails>([]);
const search = ref("");
const addUpdateProduct = ref();
const deleteProduct = ref();
let searchTimeout: ReturnType<typeof setTimeout>;

watch(search, (newSearch) => {
  if (searchTimeout) clearTimeout(searchTimeout);
  searchTimeout = setTimeout(() => {
    getProductsFromAPI(newSearch);
  }, 2000);
});

const filteredProducts = computed(() => {
  if (!search.value.trim()) return products.value;
  return products.value.filter((product) =>
    product.category.toLowerCase().includes(search.value.toLowerCase())
  );
});

onMounted(() => {
  getProducts();
});

const getProducts = async () => {
  try {
    const response = await axios.get("https://fakestoreapi.com/products");
    products.value = response.data;
  } catch (ex) {
    console.error("Error fetching products:", ex);
  }
};

const getProductsFromAPI = async (newSearch: IProductDetails) => {
  const response = await axios.get(
    `https://fakestoreapi.com/products/category/${newSearch}`
  );
};

const getProductsDetails = (newProduct: IProductDetails) => {
  products.value = [newProduct, ...products.value];
};

const onUpdate = (product: IProductDetails) => {
  addUpdateProduct.value.show(product);
};

const updateProduct = (updatedProduct: IProductDetails) => {
  console.log(updatedProduct);
  products.value = products.value.map((p) =>
    p.id === updatedProduct.id ? updatedProduct : p
  );
  console.log(  products.value )
};

const showDialog = () => {
  addUpdateProduct.value.show();
};
const onDelete = (product: IProductDetails) => {
  deleteProduct.value.openDeleteDialog(product);
};

const onDeleteProduct = (product: IProductDetails) => {
  products.value = products.value.filter((p) => p.id !== product.id);
};
</script>
