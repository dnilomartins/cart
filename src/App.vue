<script setup>
import { reactive, computed } from 'vue'
import { products } from './products.js'

const cart = reactive([])
const addItem = (product) => {
  const exists = cart.some((item) => item.product.id === product.id)
  if (exists) {
    const existingItem = cart.find((item) => item.product.id === product.id)
    existingItem.weight = existingItem.weight + product.weight
  } else {
    cart.push({
      product: { ...product },
      weight: product.weight
    })
  }
}

const removeItem = (product) => {
  const existsIndex = cart.findIndex((item) => item.product.id === product.id)
  if (existsIndex !== -1) {
    const item = cart[existsIndex]
    if (item.weight < product.weight) {
      cart.splice(existsIndex, 1)
    } else {
      item.weight = item.weight - product.weight
      if (item.weight === 0) {
        cart.splice(existsIndex, 1)
      }
    }
  }
}
const subTotal = computed(() => {
  return parseFloat(
    cart.reduce((total, item) => {
      return parseFloat((total + item.product.price * item.weight).toFixed(2))
    }, 0)
  )
})
const pricePerProduct = computed(() => {
  return (product) => {
    const cartItem = cart.find((item) => item.product.id === product.id)
    if (cartItem) {
      return parseFloat((cartItem.product.price * cartItem.weight).toFixed(2))
    } else {
      return 0
    }
  }
})
const getProductTotalWeight = computed(() => {
  return (product) => {
    const cartItem = cart.find((item) => item.product.id === product.id)
    if (cartItem) {
      return parseFloat(cartItem.weight.toFixed(2))
    } else {
      return '-'
    }
  }
})
const shipping = computed(() => {
  if (subTotal.value === 0 || subTotal.value > 400) {
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
          <div>{{ getProductTotalWeight(product) }}</div>
          <div>R${{ pricePerProduct(product) }}</div>
        </div>
      </li>
    </ul>
    <div class="calculations">
      <div>Subtotal: R${{ subTotal }}</div>
      <div>Shipping: R${{ shipping }}</div>
    </div>
  </div>
  {{ cart }}
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
