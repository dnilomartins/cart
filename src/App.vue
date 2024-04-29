<script setup>
import { reactive, computed } from 'vue'

const products = [
  {
    id: 1,
    name: 'Banana',
    price: 10,
    weith: 0.1
  },
  {
    id: 2,
    name: 'Apple',
    price: 20,
    weith: 0.16
  },
  {
    id: 3,
    name: 'Orange',
    price: 30,
    weith: 0.23
  }
]
const cart = reactive([])
const addItem = (product) => {
  const exists = cart.some((item) => item.product.id === product.id)
  if (exists) {
    const existingItem = cart.find((item) => item.product.id === product.id)
    existingItem.quantity++
  } else {
    cart.push({
      product: { ...product },
      quantity: 1
    })
  }
}
const removeItem = (product) => {
  const existsIndex = cart.findIndex((item) => item.product.id === product.id)
  if (existsIndex !== -1) {
    const item = cart[existsIndex]
    if (item.quantity > 1) {
      item.quantity--
    } else {
      cart.splice(existsIndex, 1)
    }
  }
}
const subTotal = computed(() => {
  return cart.reduce((total, item) => {
    return total + item.product.price * item.quantity
  }, 0)
})
const pricePerProduct = computed(() => {
  return (product) => {
    const cartItem = cart.find((item) => item.product.id === product.id)
    if (cartItem) {
      return cartItem.product.price * cartItem.quantity
    } else {
      return 0
    }
  }
})
const getProductQnty = computed(() => {
  return (product) => {
    const cartItem = cart.find((item) => item.product.id === product.id)
    if (cartItem) {
      return cartItem.quantity
    } else {
      return '-'
    }
  }
})
const shipping = computed(() => {
  if (subTotal.value === 0) {
    return 0
  } else if (subTotal.value > 0 && subTotal.value < 400) {
    return 1
  } else {
    return 3
  }
})
</script>

<template>
  <div class="container-item">
    <ul>
      <li v-for="product in products" :key="product.id">
        <div>{{ product.name }}</div>
        <div>
          <button @click="addItem(product)">➕</button>
          <button @click="removeItem(product)">➖</button>
        </div>
        <div class="info-product">
          <div>R${{ pricePerProduct(product) }}</div>
          <div>{{ getProductQnty(product) }}</div>
        </div>
      </li>
    </ul>
    <div class="calculations">
      <div>Subtotal: R${{ subTotal }}</div>
      <div>Shipping: R${{ shipping }}</div>
    </div>
  </div>
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

.container-item {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  align-items: center;
}

ul {
  list-style: none;
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

.calculations {
  padding: 12px;
}
</style>
