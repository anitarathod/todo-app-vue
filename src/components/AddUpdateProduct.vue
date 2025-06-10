<template>
  <div>
    <div v-if="showDialog" class="modal-overlay">
      <div class="modal-box">
        <h2>{{ update ? "Update Product" : "Add Product" }}</h2>
        <form @submit.prevent="onSubmit">
          <div class="form-group">
            <label>Product Name</label>
            <input
              v-model="productDetails.title"
              type="text"
              placeholder="Enter title for product"
              required
            />
          </div>

          <div class="form-group">
            <label>Product Description</label>
            <input
              v-model="productDetails.description"
              type="text"
              placeholder="Enter description"
              required
            />
          </div>
          <div class="form-group">
            <label>Price</label>
            <input
              v-model="productDetails.price"
              type="float"
              placeholder="Enter price"
              required
              @input="priceTouched = true"
            />
            <p v-if="priceError" style="color: red; font-size: 0.85rem">
              Please enter a valid price greater than 0.
            </p>
          </div>

          <div v-if="!update" class="form-group">
            <label>Image</label>
            <input
              type="file"
              @change="handleImageUpload"
              required
              accept="image/*"
            />
          </div>

          <div v-else class="image-preview">
            <p>Current Image Preview:</p>
            <img :src="productDetails.image" alt="Preview" height="60" />
            <label>Update Image</label>
            <input
              type="file"
              :src="productDetails.image"
              @change="handleImageUpload"
              accept="image/*"
            />
          </div>

          <div class="form-group">
            <label for="category">Select Category</label>
            <select v-model="productDetails.category" id="category" required>
              <option disabled value="">Please select one</option>
              <option
                v-for="category in categories"
                :key="category"
                :value="category"
              >
                {{ category }}
              </option>
            </select>
          </div>

          <div class="button-group">
            <button type="submit">{{ update ? "Update" : "Submit" }}</button>
            <button
              type="button"
              class="cancel-btn"
              @click="showDialog = false"
            >
              Cancel
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineEmits, defineExpose, watch } from "vue";
import axios from "axios";
import { IProductDetails } from "./src/product.ts";

const showDialog = ref(false);
const update = ref(false);

const productDetails = ref<IProductDetails>({
  id: 0,
  title: "",
  price: 0,
  description: "",
  category: "",
  image: "",
});

const emit = defineEmits(["add", "update"]);
const categories = ["men's clothing", "women's clothing", "jewelery", "other"];

const priceError = ref(false);
const priceTouched = ref(false);

watch(
  () => productDetails.value.price,
  (newPrice) => {
    if (!priceTouched.value) {
      priceError.value = false;
      return;
    }
    priceError.value = newPrice <= 0 || isNaN(newPrice);
  }
);

const show = (product?: IProductDetails) => {
  if (product) {
    update.value = true;
    productDetails.value = { ...product };
  } else {
    update.value = false;
    productDetails.value = {
      id: 0,
      title: "",
      price: 0,
      description: "",
      category: "",
      image: "",
    };
  }
  showDialog.value = true;
};

defineExpose({ show });

const handleImageUpload = (event: Event) => {
  const target = event.target as HTMLInputElement;
  const file = target.files?.[0];
  if (file) {
    productDetails.value.image = URL.createObjectURL(file);
  }
};

const onSubmit = async () => {
  try {
    const payload: IProductDetails = {
      id: productDetails.value.id,
      title: productDetails.value.title,
      price: productDetails.value.price,
      category: productDetails.value.category,
      image: productDetails.value.image,
      description: productDetails.value.description,
    };
    if (update.value) {
      await axios.put(
        `https://fakestoreapi.com/products/${payload.id}`,
        payload
      );
      emit("update", productDetails.value);
    } else {
      await axios.post(`https://fakestoreapi.com/products`, payload);
      emit("add", productDetails.value);
    }
    showDialog.value = false;
  } catch (ex) {
    console.error("Error submitting product:", ex);
  }
};
</script>
