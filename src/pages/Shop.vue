<template>
  <div>
    <v-app-bar app color="white" flat>
      <v-toolbar-title class="white--text">K-Bite Lounge</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-badge
        overlap
        color="red"
        :content="totalCartItems"
        v-if="totalCartItems > 0"
        class="mr-4"
      >
        <v-btn class="cart-button" icon @click="openCart">
          <v-icon>mdi-cart</v-icon>
        </v-btn>
      </v-badge>
    </v-app-bar>

    <v-container id="cont" class="shop-container" fluid>
      <h1 class="os text-center my-4">Products</h1>
      <v-row v-if="products.length > 3">
        <v-carousel cycle show-arrows :items-to-show="1" color="white">
          <v-carousel-item v-for="(group, index) in chunkedProducts" :key="index">
            <v-row class="d-flex justify-center">
              <v-col
                v-for="product in group"
                :key="product.id"
                class="product-card mt-5 mx-5"
                cols="3"
              >
                <v-img :src="product.image" class="product-image" />
                <div class="product-details">
                  <h2>{{ product.title }}</h2>
                  <p>Qty: {{ product.quantity }}</p>
                  <p>Price: {{ product.price }}</p>
                  <button @click="addToCart(product.id)">
                    <v-icon>mdi-cart</v-icon> Add to Cart
                  </button>
                </div>
              </v-col>
            </v-row>
          </v-carousel-item>
        </v-carousel>
      </v-row>

      <v-row v-else>
        <v-col
          v-for="product in products"
          :key="product.id"
          class="product-card mt-5 mx-5"
          cols="3"
        >
          <v-img :src="product.image" class="product-image" />
          <div class="product-details">
            <h2>{{ product.title }}</h2>
            <p>Qty: {{ product.quantity }}</p>
            <p>Price: {{ product.price }}</p>
            <button @click="addToCart(product.id)">
              <v-icon>mdi-cart</v-icon> Add to Cart
            </button>
          </div>
        </v-col>
      </v-row>

      <v-dialog v-model="cartDialog" max-width="800px">
        <v-card>
          <v-card-title>
            <h2 class="os text-center my-5">Your Cart</h2>
          </v-card-title>
          <v-card-text>
            <v-row class="d-flex justify-center mb-5">
              <v-col
                v-for="item in cartItems"
                :key="item.id"
                class="product-card mt-5 mx-5"
                cols="3"
              >
                <v-img :src="item.product.image" class="product-image" />
                <div class="product-details">
                  <h2>{{ item.product.title }}</h2>
                  <p>Price: {{ item.product.price }}</p>
                  <button @click="removeFromCart(item.id, item.product.id)">
                    <v-icon>mdi-delete</v-icon> Remove
                  </button>
                </div>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-actions>
            <v-btn @click="closeCart" color="primary">Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from "vue";
import { supabase } from "@/lib/supabase";

const products = ref([]);
const cartItems = ref([]);
const totalCartItems = ref(0);
const cartDialog = ref(false);

const openCart = () => {
  cartDialog.value = true;
};

const closeCart = () => {
  cartDialog.value = false;
};

const chunkedProducts = computed(() => {
  const result = [];
  for (let i = 0; i < products.value.length; i += 3) {
    result.push(products.value.slice(i, i + 3));
  }
  return result;
});

const fetchProducts = async () => {
  const { data, error } = await supabase.from("products").select("*");
  if (error) {
    console.error("Error fetching products:", error.message);
    return;
  }
  products.value = data;
};

const fetchCart = async () => {
  const userId = localStorage.getItem("userId");
  if (!userId) {
    console.warn("No user ID found");
    return;
  }
  const { data, error } = await supabase
    .from("inventories")
    .select("*, product:products(*)")
    .eq("user_id", userId);
  if (error) {
    console.error("Error fetching cart:", error.message);
    return;
  }
  cartItems.value = data || [];
  totalCartItems.value = cartItems.value.length;
};

const addToCart = async (productId) => {
  const userId = localStorage.getItem("userId");
  if (!userId) {
    console.warn("User ID missing");
    return;
  }
  const { data: productData, error } = await supabase
    .from("products")
    .select("quantity")
    .eq("id", productId)
    .single();

  if (error || !productData || productData.quantity <= 0) {
    console.error("Product not available");
    return;
  }

  await supabase
    .from("products")
    .update({ quantity: productData.quantity - 1 })
    .eq("id", productId);
  await supabase
    .from("inventories")
    .insert([{ user_id: userId, product_id: productId }]);
  await fetchProducts();
  await fetchCart();
};

const removeFromCart = async (cartId, productId) => {
  const { data: productData, error } = await supabase
    .from("products")
    .select("quantity")
    .eq("id", productId)
    .single();

  if (error || !productData) {
    console.error("Error retrieving product details");
    return;
  }

  await supabase
    .from("products")
    .update({ quantity: productData.quantity + 1 })
    .eq("id", productId);
  await supabase.from("inventories").delete().eq("id", cartId);
  await fetchProducts();
  await fetchCart();
};

onMounted(() => {
  fetchProducts();
  fetchCart();
});
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Comforter&display=swap");

.os {
  font-family: "Fira Sans", sans-serif;
  font-size: 3rem;
}
#cont {
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #3854b8, #ca9af2);
  display: flex;
  align-items: center; /* Vertical alignment */
  justify-content: center; /* Horizontal alignment */
  flex-direction: column; /* To align items in a column */
}

.product-card {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.product-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}
.product-details {
  text-align: center;
}
button {
  background-color: #b1860e;
  color: white;
  border: none;
  padding: 8px 12px;
  cursor: pointer;
  border-radius: 4px;
}
button:hover {
  background-color: #d4a517;
}
</style>

