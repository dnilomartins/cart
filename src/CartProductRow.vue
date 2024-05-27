<script setup>
import { defineProps, computed } from 'vue'

const props = defineProps({
  cart: {
    type: Array,
    required: true
  },
  product: {
    type: Object,
    required: true
  }
})

const productTotalWeight = computed(() => {
  const cartItem = props.cart.find((item) => item.product.id === props.product.id)
  return cartItem ? cartItem.weight : 0
})

const pricePerProduct = computed(() => {
  const cartItem = props.cart.find((item) => item.product.id === props.product.id)
  return cartItem && cartItem.weight > 0 ? cartItem.product.price * cartItem.weight : 0
})
</script>

<template>
  <li class="product-row">
    <div>{{ props.product.name }}</div>
    <div>
      <button @click="$emit('add')">➕</button>
      <button @click="$emit('remove')">➖</button>
    </div>
    <div class="info-product">
      <div>{{ productTotalWeight.toFixed(1) }}</div>
      <div>R${{ pricePerProduct.toFixed(2) }}</div>
    </div>
  </li>
</template>

<style scoped>
.product-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  align-items: center;
}

.info-product {
  display: flex;
  gap: 8px;
}
</style>
