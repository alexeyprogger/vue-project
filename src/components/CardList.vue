<script setup>
import Card from './Card.vue'

defineProps({
  items: Array,
  isFav: Boolean
})

const emit = defineEmits(['addToFavourite', 'addToCart'])
</script>

<template>
  <div class="grid grid-cols-2 gap-5" v-auto-animate >
    <Card
      v-for="item in items"
      :key="item.id"
      :id="item.id"
      :title="item.title"
      :imageUrl="item.imageUrl"
      :price="item.price"
      :isAdded="item.isAdded"
      :isFavourite="item.isFavourite"
      :onClickAdd="isFav ? null : () => emit('addToCart', item)"
      :onClickFavourite="isFav ? null : () => emit('addToFavourite', item)"
      />
    </div>
    <!-- # TODO: onСlick -> вызов функции с модальным окном  -->
    <!-- # TODO:   -->
</template>