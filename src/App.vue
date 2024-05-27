<script setup>
import { computed, ref } from 'vue'
import { products } from './products.js'
import { coupons } from './coupons.js'
import CartProductRow from './CartProductRow.vue'

const cart = ref([])
const couponCodeInput = ref('')
const currentCoupon = ref(null)

const addItem = (product) => {
  const cartItem = cart.value.find((item) => item.product.id === product.id)

  if (cartItem) {
    cartItem.weight = cartItem.weight + product.weight
    return
  }

  cart.value.push({
    product: { ...product },
    weight: product.weight
  })
}

const removeItem = (product) => {
  const cartItem = cart.value.findIndex((item) => item.product.id === product.id)

  if (cartItem === -1) {
    return
  }

  const item = cart.value[cartItem]

  if (item.weight < product.weight) {
    return
  }

  item.weight -= product.weight

  if (currentCoupon.value?.type === 'fixed' && subTotal.value < currentCoupon.value.amount) {
    currentCoupon.value = null
  }

  if (item.weight !== 0) {
    return
  }

  cart.value.splice(cartItem, 1)
}

const subTotal = computed(() => {
  return cart.value.reduce((total, item) => {
    return total + item.product.price * item.weight
  }, 0)
})

const discount = computed(() => {
  if (currentCoupon.value === null) return 0

  if (currentCoupon.value.type === 'percent') {
    const percentDiscount = currentCoupon.value.amount
    const discountAmount = subTotal.value * (percentDiscount / 100)
    return discountAmount
  }

  if (currentCoupon.value.type === 'fixed') {
    return currentCoupon.value.amount
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

const total = computed(() => {
  return subTotal.value + shipping.value - discount.value
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
      <CartProductRow
        v-for="product in products"
        :key="product.id"
        :cart="cart"
        :product="product"
        @add="addItem(product)"
        @remove="removeItem(product)"
      />
    </ul>
    <div class="calculations">
      <div>
        <input type="text" v-model="couponCodeInput" placeholder="Digite seu cupom" />
        <button @click="applyCoupon">Aplicar</button>
        <button @click="removeCoupon">Remover</button>
      </div>
      <div>Subtotal: R${{ subTotal.toFixed(2) }}</div>
      <div v-if="currentCoupon?.type === 'free_shipping'">Shipping: Frete Grátis</div>
      <div v-else>Shipping: R${{ shipping }}</div>
      <div>
        <div v-if="currentCoupon?.type === 'free_shipping'">Desconto: -</div>
        <div v-else>Desconto: R${{ discount.toFixed(2) }}</div>
      </div>
      <div>Total: R${{ total.toFixed(2) }}</div>
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
</style>
