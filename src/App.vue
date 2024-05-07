<script setup>
import { ref, watch, provide, computed, onMounted } from 'vue'
import Header from './components/Header.vue'
import Drawer from './components/Drawer.vue'

const cart = ref([])
const drawerOpen = ref(false)

const totalPrice = computed(() => cart.value.reduce((acc, item) => acc + item.price, 0))

const closeDrawer = () => {
  drawerOpen.value = false
}
const openDrawer = () => {
  drawerOpen.value = true
}

const addToCart = (item) => {
  cart.value.push(item)
}
const removeFromCart = (item) => {
  console.log(cart.value.map(e => e.id).indexOf(item.id));
  console.log('С помощью find(): ', cart.value.find(el => el.id === item.id)) // find вернёт объект (не число)
  console.log('С помощью map(): ', cart.value.map(e => e.id).indexOf(item.id)) // т.о. вернём число для splice
  let b = cart.value.splice(cart.value.map(e => e.id).indexOf(item.id), 1)
  item.isAdded = false
}

watch(
  cart,
  () => {
    localStorage.setItem('cart', JSON.stringify(cart.value))
  },
  { deep: true }
)

onMounted(async ()=>{
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []
})

provide('cart', {
  closeDrawer,
  openDrawer,
  cart,
  addToCart,
  removeFromCart
})
</script>

<template>
  <Drawer v-if="drawerOpen" :total-price="totalPrice" />
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Header :total-price="totalPrice" @open-drawer="openDrawer" />

    <div class="p-10">
      <router-view></router-view>
    </div>
  </div>
</template>
