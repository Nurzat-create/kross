<script setup>
import { onMounted, reactive, provide, ref, watch } from 'vue'
import axios from 'axios'
import Header from './components/Header.vue'
import CartList from './components/CartList.vue'
import Drawer from './components/Drawer.vue'

const items = ref([])
const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
})
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}
const onChangeSearchInput = (event) => {
  filters.searchQuery = event.target.value
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://9e7c08c12bebf8de.mokky.dev/favorites`)
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.parentId === item.id)

      if (!favorite) {
        return item
      }
      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
    console.log(items.value)
  } catch (er) {
    console.log(err)
  }
}
const addToFavorite = async (item) => {
  item.isFavorite = true

  console.log(item)
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }
    const { data } = await axios.get(`https://9e7c08c12bebf8de.mokky.dev/items`, {
      params
    })
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      isAddded: false
    }))
  } catch (er) {
    console.log(err)
  }
}

onMounted(async () => {
  await fetchItems()
  await fetchFavorites()
})
watch(filters, fetchItems)

provide('addToFavorite', addToFavorite)
</script>

<template>
  <!-- <Drawer /> -->
  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-14">
    <Header />
    <div class="p-10">
      <div class="flex justify-between items-center">
        <h2 class="text-3xl font-bold mb-8">Все Кроссовки</h2>

        <div class="flex gap-4">
          <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
            <option value="name">По названию</option>
            <option value="price">По цене (дешевые)</option>
            <option value="-price">По цене (дорогие)</option>
          </select>

          <div class="relative">
            <img class="absolute left-4 top-3" src="/search.svg" alt="" />
            <input
              @change="onChangeSearchInput"
              class="border rounded-md py-1.5 pl-11 pr-4 outline-none focus:border-gray-400"
              placeholder="Поиск..."
            />
          </div>
        </div>
      </div>
      <div class="mt-10">
        <CartList :items="items" />
      </div>
    </div>
  </div>
</template>
