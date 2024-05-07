<script setup>
import { reactive, inject, watch, ref, onMounted } from 'vue'
import CardList from '../components/CardList.vue'
import axios from 'axios'
import debounce from 'lodash.debounce'

const { cart, addToCart, removeFromCart } = inject('cart')

const items = ref([])

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})

const onClickAddPlus = (item) => {
  console.log(!item.isAdded ? 'Добавлен': 'Не добавлен')
  if (!item.isAdded) {
    addToCart(item)
    item.isAdded = true
  } else {
    console.log('Из Home: item.isAdded = ', item.isAdded)
    removeFromCart(item)
  }
}
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
  console.log(filters.sortBy)
}
const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 800)

const addToFavourite = async (item) => {
  try {
    item.isFavourite = !item.isFavourite
    console.log(item.isFavourite)
    if (item.isFavourite) {
      const obj = {
        item_id: item.id
      }
      const { data } = await axios.post('https://21d6dab34fae5924.mokky.dev/favourites', obj)
      item.favouriteId = data.id
    } else {
      await axios.delete(`https://21d6dab34fae5924.mokky.dev/favourites/${item.favouriteId}`)
      item.favouriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

const fetchFavourites = async () => {
  try {
    const { data: favourites } = await axios.get('https://21d6dab34fae5924.mokky.dev/favourites')
    items.value = items.value.map((item) => {
      const favourite = favourites.find((favourite) => favourite.item_id === item.id)

      if (!favourite) {
        return item
      }

      return {
        ...item,
        isFavourite: true,
        favouriteId: favourite.id
      }
    })
  } catch (err) {
    console.log(err)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }
    const { data } = await axios.get('https://21d6dab34fae5924.mokky.dev/items', {
      params
    })
    items.value = data.map((obj) => ({
      ...obj,
      isFavourite: false,
      isAdded: false,
      favouriteId: null
    }))
  } catch (err) {
    console.log(err)
  }
}

onMounted(async () => {
  await fetchItems()
  await fetchFavourites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

watch(filters, ()=> {
  fetchItems()
  fetchFavourites()
})
</script>
<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все экскурсии</h2>

    <div class="flex gap-4 max-md:flex max-lg:flex-col">
      <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
        <option value="title">По названию</option>
        <option value="price">Дешёвые</option>
        <option value="-price">Дорогие</option>
      </select>
      <div class="relative">
        <img class="absolute left-3 top-3" src="/search.svg" />
        <input
          @input="onChangeSearchInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          placeholder="Поиск"
        />
      </div>
    </div>
  </div>
  <div class="mt-10">
    <CardList :items="items" @add-to-favourite="addToFavourite" @add-to-cart="onClickAddPlus" />
  </div>
</template>
