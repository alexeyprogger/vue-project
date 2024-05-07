<script setup>
import { ref, onMounted, inject } from 'vue'
import axios from 'axios'
import CardList from '../components/CardList.vue'

const favourites = ref([])
const { cart } = inject('cart')

onMounted(async () => {
  try {
    const { data } = await axios.get(
      'https://21d6dab34fae5924.mokky.dev/favourites?_relations=items'
    )
    favourites.value = data.map((obj) => obj.item)
  } catch (err) {
    console.log(err)
  }
})
</script>

<template>
  <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>
  <CardList :items="favourites" isFav />
</template>
