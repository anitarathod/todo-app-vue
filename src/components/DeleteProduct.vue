<template>
  <div v-if="showDeleteDialog" class="dialog-overlay">
    <div class="dialog">
      <h3>Confirm Deletion</h3>
      <p>
        Are you sure you want to delete
        <strong>{{ productToDelete?.title }}</strong
        >?
      </p>
      <div class="dialog-actions">
        <button class="btn-cancel" @click="cancelDelete">Cancel</button>
        <button class="btn-delete" @click="confirmDelete">Delete</button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, defineExpose, defineEmits } from "vue";
import { IProductDetails } from "./src/product.ts";
import axios from "axios";

const emit = defineEmits(["delete"]);

const showDeleteDialog = ref(false);
const productToDelete = ref<IProductDetails>(null);

const openDeleteDialog = (product: IProductDetails) => {
  productToDelete.value = product;
  showDeleteDialog.value = true;
};

const cancelDelete = () => {
  showDeleteDialog.value = false;
  productToDelete.value = null;
};

const confirmDelete = async () => {
  if (!productToDelete.value) return;
  try {
    await axios.delete(
      `https://fakestoreapi.com/products/${productToDelete.value.id}`
    );
    // alert(`Product ${productToDelete.value.title} deleted successfully`);
    emit("delete", productToDelete.value);
    cancelDelete();
  } catch (error) {
    console.error("Failed to delete product:", error);
  }
};
defineExpose({ openDeleteDialog });
</script>
