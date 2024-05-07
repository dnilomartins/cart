<script setup>
import { computed, ref } from 'vue'
import { products } from './products.js'
import { coupons } from './coupons.js'

const cart = ref([])
const couponCode = ref('')
const currentCoupon = ref(null)

const addItem = (product) => {
  const exists = cart.value.some((item) => item.product.id === product.id)
  if (exists) {
    const existingItem = cart.value.find((item) => item.product.id === product.id)
    existingItem.weight = existingItem.weight + product.weight
  } else {
    cart.value.push({
      product: { ...product },
      weight: product.weight
    })
  }
}

const removeItem = (product) => {
  const existsIndex = cart.value.findIndex((item) => item.product.id === product.id)
  if (existsIndex !== -1) {
    const item = cart.value[existsIndex]
    if (item.weight < product.weight) {
      cart.value.splice(existsIndex, 1)
    } else {
      item.weight = item.weight - product.weight
      if (item.weight === 0) {
        cart.value.splice(existsIndex, 1)
      }
    }
    if (
      currentCoupon.value &&
      currentCoupon.value.code === 'FOO' &&
      subTotal.value >= currentCoupon.value.amount
    ) {
      currentCoupon.value = null
    }
  }
}

const subTotal = computed(() => {
  return parseFloat(
    cart.value.reduce((total, item) => {
      return parseFloat((total + item.product.price * item.weight).toFixed(2))
    }, 0)
  )
})

const pricePerProduct = computed(() => {
  return (product) => {
    const cartItem = cart.value.find((item) => item.product.id === product.id)
    if (cartItem) {
      return parseFloat((cartItem.product.price * cartItem.weight).toFixed(2))
    } else {
      return 0
    }
  }
})

const getProductTotalWeight = computed(() => {
  return (product) => {
    const cartItem = cart.value.find((item) => item.product.id === product.id)
    if (cartItem) {
      return parseFloat(cartItem.weight.toFixed(2))
    } else {
      return '-'
    }
  }
})

const discount = computed(() => {
  if (currentCoupon.value === null) return 0
  if (currentCoupon.value.type === 'percent') {
    const percentDiscount = parseFloat(currentCoupon.value.amount)
    const discountAmount = subTotal.value * (percentDiscount / 100)
    return discountAmount.toFixed(2)
  } else if (currentCoupon.value.type === 'fixed') {
    return parseFloat(currentCoupon.value.amount)
  }
  return 0
})

const shipping = computed(() => {
  const COST_PER_5KG = 7
  const KG_THRESHOLD = 10
  const WEIGHT_INCREMENT = 5
  const subtotal = subTotal.value
  const totalWeight = cart.value.reduce((total, item) => total + item.weight, 0)

  console.log({ cart, subtotal, currentCoupon })
  if (
    cart.value.length === 0 ||
    subtotal > 400 ||
    (currentCoupon.value?.type === 'free_shipping' && subtotal >= 300.5)
  ) {
    return 0
  } else {
    let shippingCost = 30
    const additionalWeight = totalWeight - KG_THRESHOLD

    if (additionalWeight > 0) {
      const extraCost = Math.ceil(additionalWeight / WEIGHT_INCREMENT) * COST_PER_5KG
      shippingCost += extraCost
    }

    return shippingCost
  }
})

const applyCoupon = () => {
  if (couponCode.value === '') {
    alert('Por favor, insira um cupom válido.')
    return
  }
  if (currentCoupon.value) {
    alert('Já existe um cupom aplicado')
    return
  }
  const coupon = coupons.find((c) => c.code === couponCode.value)
  if (!coupon) {
    alert('O cupom inserido não é válido.')
    return
  }
  if (cart.value.length === 0) {
    alert('Seu carrinho está vazio')
    return
  }
  if (subTotal.value < coupon.amount && coupon.type === 'fixed') {
    alert('Subtotal é inferior ao valor de desconto')
    return
  }
  if (coupon.type === 'free_shipping' && subTotal.value > 300.5) {
    alert('O cupom de frete grátis não pode ser aplicado.')
    return
  }
  currentCoupon.value = coupon
}

const removeCoupon = () => {
  currentCoupon.value = null
}
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
      <div>
        <input type="text" v-model="couponCode" placeholder="Digite seu cupom" />
        <button @click="applyCoupon">Aplicar</button>
        <button @click="removeCoupon">Remover</button>
      </div>
      <div>Subtotal: R${{ subTotal }}</div>
      <div>Shipping: R${{ shipping }}</div>
      <div>Desconto: R${{ discount }}</div>
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
