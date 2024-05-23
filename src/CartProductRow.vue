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

const getProductTotalWeight = computed(() => {
  return (product) => {
    const cartItem = props.cart.find((item) => item.product.id === product.id)
    if (cartItem) {
      return cartItem.weight
    }
    return 0
  }
})

const pricePerProduct = computed(() => {
  return (product) => {
    const cartItem = props.cart.find((item) => item.product.id === product.id)
    if (cartItem && cartItem.weight > 0) {
      return cartItem.product.price * cartItem.weight
    }
    return 0
  }
})
</script>

<template>
  <li>
    <div>{{ props.product.name }}</div>
    <div>
      <button @click="$emit('add')">➕</button>
      <button @click="$emit('remove')">➖</button>
    </div>
    <div class="info-product">
      <div v-if="getProductTotalWeight(props.product) !== 0">
        {{ getProductTotalWeight(props.product).toFixed(1) }}
      </div>
      <div v-else>-</div>
      <div v-if="pricePerProduct(props.product) !== 0">
        R${{ pricePerProduct(props.product).toFixed(2) }}
      </div>
      <div v-else>R$0</div>
    </div>
  </li>
</template>

<style scoped>
* {
  padding: 0;
  margin: auto;
  box-sizing: border-box;
}

body {
  padding: 0;
  margin: 0;
}

li {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  align-items: center;
}

.info-product {
  display: flex;
  gap: 8px;
}
</style>
