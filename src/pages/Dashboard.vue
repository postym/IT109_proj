<template>
  <div class="dashboard-container">
    <!-- Admin Icon -->
    <div class="admin-icon-container">
      <v-avatar size="40">
        <v-img src="/src/assets/grilledbalut.jpg" alt="Admin Icon" />
      </v-avatar>
    </div>


    <!-- Main Content -->
    <div class="main-content">
      
      <!-- Products Table -->
      <br /><br /><br /><br /><br />
      <ProductsTable :products="products" :headers="headers" />

      <!-- Add Product Button -->
      <div class="add-product-btn-container text-center pa-4">
        <v-btn
          class="text-none font-weight-regular"
          prepend-icon="mdi-plus"
          variant="tonal"
          color="black"
          @click="dialog = true"
        >
          Add Product
        </v-btn>
      </div>

      <!-- Dialog for Adding/Editing Product -->
      <v-dialog v-model="dialog" max-width="600">
  <v-card class="light-gray-background">
    <v-card-title class="custom-card-title">Add New Product Details</v-card-title>
    <v-card-text>
      <v-row dense>
        <!-- Single Column for Form Fields -->
        <v-col cols="12">
          <v-text-field
            v-model="product.title"
            label="Product Title*"
            color="gray"
            class="mb-4"
            outlined
            required
          ></v-text-field>
        </v-col>

        <v-col cols="12">
          <v-file-input
            v-model="productImage"
            :rules="rules"
            accept="image/png, image/jpeg, image/bmp"
            label="Product Image*"
            color="gray"
            class="mb-4"
            placeholder="Upload Image"
            prepend-icon="mdi-camera"
            outlined
            required
          ></v-file-input>
        </v-col>

        <v-col cols="12">
          <v-text-field
            v-model="product.price"
            label="Product Price*"
            type="number"
            color="gray"
            class="mb-4"
            outlined
            required
          ></v-text-field>
        </v-col>

        <v-col cols="12">
          <v-text-field
            v-model="product.quantity"
            label="Product Quantity*"
            type="number"
            color="gray"
            outlined
            required
          ></v-text-field>
        </v-col>
      </v-row>
      <small class="text-caption text-medium-emphasis">*indicates required field</small>
    </v-card-text>
    <v-divider></v-divider>
    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn variant="plain" color="black" @click="dialog = false">Close</v-btn>
      <v-btn variant="tonal" color="blue" @click="saveProduct">
        Save
      </v-btn>
    </v-card-actions>
  </v-card>
</v-dialog>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { supabase } from "@/lib/supabase";
import ProductsTable from "@/components/ProductsTable.vue";

const search = ref("");
const products = ref([]);
const dialog = ref(false);
const product = ref({
  title: "",
  image: "",
  price: 0,
  quantity: 0,
});
const productImage = ref(null);

const headers = ref([
  { title: "Product Title", key: "title" },
  { title: "Image", key: "image", sortable: false },
  { title: "Price", key: "price" },
  { title: "Stock", key: "quantity" },
  { title: "Actions", key: "actions", sortable: false },
]);

const saveProduct = async () => {
  try {
    let imageUrl = "";

    if (productImage.value) {
      const fileName = `public/${Date.now()}-${productImage.value.name}`;
      const { error: uploadError } = await supabase.storage
        .from("products")
        .upload(fileName, productImage.value);

      if (uploadError) throw uploadError;

      imageUrl = `${
        supabase.storage.from("products").getPublicUrl(fileName).data.publicUrl
      }`;
    }

    const { error } = await supabase.from("products").insert([
      {
        title: product.value.title,
        image: imageUrl,
        price: product.value.price,
        quantity: product.value.quantity,
      },
    ]);
    if (error) throw error;

    fetchProducts();
  } catch (error) {
    console.error("Error saving product:", error);
  } finally {
    dialog.value = false;
  }
};

const fetchProducts = async () => {
  try {
    const { data, error } = await supabase
      .from("products")
      .select("title, image, price, quantity");
    if (error) throw error;

    products.value = data.map((item) => ({
      title: item.title,
      image: item.image,
      price: item.price || 0.0,
      quantity: item.quantity || 0,
    }));
  } catch (error) {
    console.error("Error fetching products:", error);
  }
};

onMounted(() => {
  fetchProducts();
});
</script>

<style scoped>
.dashboard-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
  position: relative;
}

.admin-icon-container {
  position: absolute;
  top: 16px;
  right: 16px;
  z-index: 10;
}

.light-gray-background {
  background: linear-gradient(135deg, #3854b8, #ca9af2);
}

.main-content {
  flex-grow: 1;
  overflow: auto;
  padding: 16px;
  background: linear-gradient(135deg, #3854b8, #ca9af2);
}

.search-bar-container {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 16px;
}

.search-bar {
  max-width: 300px;
}

.add-product-btn-container {
  margin-top: 16px;
}

.custom-card-title {
  background-color: rgb(61, 68, 200);
  color: white;
  padding: 16px;
  border-radius: 4px;
}

/* Adjust Product Table Styles */
.products-table tr {
  background: transparent !important;
}

.products-table .edit-icon {
  color: gray !important;
}

.products-table .delete-icon {
  color: red !important;
}
</style>
