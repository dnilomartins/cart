<script setup>
import { defineProps } from 'vue'

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

const getProductTotalWeight = (product) => {
  const cartItem = props.cart.find((item) => item.product.id === product.id)
  if (cartItem) {
    return cartItem.weight
  }
  return 0
}

const GetPricePerProduct = (product) => {
  const cartItem = props.cart.find((item) => item.product.id === product.id)
  if (cartItem && cartItem.weight > 0) {
    return cartItem.product.price * cartItem.weight
  }
  return 0
}
</script>

<template>
  <li class="product-row">
    <div>{{ props.product.name }}</div>
    <div>
      <button @click="$emit('add')">➕</button>
      <button @click="$emit('remove')">➖</button>
    </div>
    <div class="info-product">
      <div>
        {{ getProductTotalWeight(props.product).toFixed(1) }}
      </div>
      <div>R${{ GetPricePerProduct(props.product).toFixed(2) }}</div>
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
