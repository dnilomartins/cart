<script setup>
import { computed, ref } from 'vue'
import { products } from './products.js'
import { coupons } from './coupons.js'

const cart = ref([])
const couponCodeInput = ref('')
const currentCoupon = ref(null)

const addItem = (product) => {
  const exists = cart.value.find((item) => item.product.id === product.id)

  if (exists) {
    exists.weight = exists.weight + product.weight
    return
  }

  cart.value.push({
    product: { ...product },
    weight: product.weight
  })
}

const removeItem = (product) => {
  const existsIndex = cart.value.findIndex((item) => item.product.id === product.id)

  if (existsIndex !== -1) {
    const item = cart.value[existsIndex]

    if (item.weight >= product.weight) {
      item.weight -= product.weight
    }

    if (item.weight === 0) {
      cart.value.splice(existsIndex, 1)
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

const getProductTotalWeight = computed(() => {
  return (product) => {
    const cartItem = cart.value.find((item) => item.product.id === product.id)
    if (cartItem) {
      return cartItem.weight
    }
    return 0
  }
})

const pricePerProduct = computed(() => {
  return (product) => {
    const cartItem = cart.value.find((item) => item.product.id === product.id)
    if (cartItem && cartItem.weight > 0) {
      return cartItem.product.price * cartItem.weight
    }
    return 0
  }
})

const discount = computed(() => {
  if (currentCoupon.value !== null) {
    if (currentCoupon.value.type === 'percent') {
      const percentDiscount = currentCoupon.value.amount
      const discountAmount = subTotal.value * (percentDiscount / 100)
      return discountAmount
    }

    if (currentCoupon.value.type === 'fixed') {
      return currentCoupon.value.amount
    }
  }

  return 0
})

const totalWeight = computed(() => {
  return cart.value.reduce((total, item) => total + item.weight, 0)
})

const BASE_SHIPPING_COST = 30
const COST_PER_5KG = 7
const KG_THRESHOLD = 10
const WEIGHT_INCREMENT = 5
const FREE_SHIPPING_THRESHOLD = 400
const FREE_SHIPPING_SUBTOTAL_THRESHOLD = 300.5

const shipping = computed(() => {
  const subtotal = subTotal.value

  if (
    cart.value.length === 0 ||
    subtotal > FREE_SHIPPING_THRESHOLD ||
    (currentCoupon.value?.type === 'free_shipping' && subtotal >= FREE_SHIPPING_SUBTOTAL_THRESHOLD)
  ) {
    return 0
  }

  let shippingCost = BASE_SHIPPING_COST
  const additionalWeight = totalWeight.value - KG_THRESHOLD

  if (additionalWeight > 0) {
    const extraCost = Math.ceil(additionalWeight / WEIGHT_INCREMENT) * COST_PER_5KG
    shippingCost += extraCost
  }

  if (currentCoupon.value?.type === 'free_shipping') {
    return 0
  }

  return shippingCost
})

const applyCoupon = () => {
  if (couponCodeInput.value === '') {
    alert('Por favor, insira um cupom válido.')
    return
  }

  if (currentCoupon.value) {
    alert('Já existe um cupom aplicado')
    return
  }

  const coupon = coupons.find((c) => c.code === couponCodeInput.value)

  if (!coupon) {
    alert('O cupom inserido não é válido.')
    return
  }

  if (cart.value.length === 0) {
    alert('Seu carrinho está vazio')
    return
  }

  if (coupon.type === 'free_shipping' && subTotal.value < FREE_SHIPPING_SUBTOTAL_THRESHOLD) {
    alert('Para usar esse cupom seu subtotal deverá ser maior ou igual a R$300,50')
    return
  }

  if (subTotal.value < coupon.amount && coupon.type === 'fixed') {
    alert('Subtotal é inferior ao valor de desconto')
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
          <div v-if="getProductTotalWeight(product) !== 0">
            {{ getProductTotalWeight(product).toFixed(1) }}
          </div>
          <div v-else>-</div>
          <div v-if="pricePerProduct(product) !== 0">
            R${{ pricePerProduct(product).toFixed(2) }}
          </div>
          <div v-else>R$0</div>
        </div>
      </li>
    </ul>
    <div class="calculations">
      <div>
        <input type="text" v-model="couponCodeInput" placeholder="Digite seu cupom" />
        <button @click="applyCoupon">Aplicar</button>
        <button @click="removeCoupon">Remover</button>
      </div>
      <div>Subtotal: R${{ subTotal }}</div>
      <div v-if="currentCoupon && currentCoupon.type === 'free_shipping'">
        Shipping: Frete Grátis
      </div>
      <div v-else>Shipping: R${{ shipping }}</div>
      <div>
        <div v-if="currentCoupon && currentCoupon.type === 'free_shipping'">Desconto: -</div>
        <div v-else>Desconto: R${{ discount.toFixed(2) }}</div>
      </div>
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
