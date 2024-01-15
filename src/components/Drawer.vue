<script setup>
import axios from 'axios'
import { ref, computed, inject } from 'vue'
import DrawerHead from './DrawerHead.vue'
import CartItemList from './CartItemList.vue'
import InfoEmpty from './infoEmpty.vue'

const props = defineProps({
  totalPrice: Number
})

const { cart, closeDrawer } = inject('cart')
const isCreating = ref(false)
const orderId = ref(null)

const buttonDisabled = computed(() => (isCreating.value ? true : false))

const createOrder = async () => {
  try {
    isCreating.value = true
    const { data } = await axios.post(`https://21d6dab34fae5924.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: props.totalPrice.value
    })
    cart.value = []
    orderId.value = data.id
  } catch (err) {
    console.log(err)
  } finally {
    isCreating.value = false
  }
}
</script>

<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-98 h-full fixed right-0 top-0 z-20 p-8 overflow-auto">
    <DrawerHead />

    <div v-if="!totalPrice || orderId" class="flex h-full items-center">
      <InfoEmpty
        v-if="!totalPrice && !orderId"
        title="Корзина пустая"
        description="Добавьте хоть что-то"
        image-url="/package-icon.png"
      />

      <InfoEmpty
        v-if="orderId"
        title="Заказ оформлен"
        :description="`Ваш заказ №${orderId} находится в обработке. Счастливого экскурсионного времяпрепровождения!`"
        image-url="/order-success-icon.png"
      />
    </div>

    <div v-else>
      <CartItemList />
      <div class="flex flex-col gap-4 mt-7">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ totalPrice }} руб.</b>
        </div>
        <button
          :disabled="buttonDisabled"
          @click="createOrder"
          class="mt-4 bg-lime-500 w-full disabled:bg-slate-300 cursor-pointer rounded-xl py-3 text-white hover:bg-lime-600 transition active:bg-lime-700"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>
