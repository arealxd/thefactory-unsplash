<script setup lang="ts">
import HeaderComponent from '@/components/HeaderComponent.vue'
import { ref } from 'vue'
import axios from 'axios'
import type { ImageResponse } from '@/types/Image'

const loading = ref(true)
const notFound = ref(false)
const photos = ref<ImageResponse[]>()

const setSearchImages = (search: ImageResponse[]) => {
  if (search) {
    photos.value = search
    loading.value = false
    console.log(photos.value.length)

    if (photos.value.length === 0) {
      notFound.value = true
    }
    if (photos.value.length === 9) {
      notFound.value = false
    }
  }
}

axios
  .get('https://api.unsplash.com/photos/random', {
    params: {
      count: 9
    },
    headers: {
      Authorization: `Client-ID ${import.meta.env.VITE_ACCESS_KEY}`
    }
  })
  .then((response) => {
    photos.value = response.data
    notFound.value = false
    loading.value = false
  })
  .catch((error) => {
    console.error(error)
  })
</script>

<template>
  <HeaderComponent @search="setSearchImages" />
  <div class="container">
    <div class="loader" v-if="loading">
      <p>Загрузка...</p>
    </div>
    <div class="loader" v-else-if="notFound">
      <p>Изображений не найдены</p>
    </div>
    <div class="images" v-else>
      <router-link v-for="i in photos" :key="i.id" :to="'/details/' + i.id">
        <img class="image" :src="i.urls.regular" alt="" />
      </router-link>
    </div>
  </div>
</template>

<style scoped lang="scss">
.loader {
  display: flex;
  justify-content: center;
  margin-top: 50px;
  p {
    font-weight: 700;
    font-size: 30px;
    color: #000000;
  }
}
.images {
  margin-top: 100px;
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 30px;
  margin-bottom: 200px;
  .image {
    width: 100%;
    height: 100%;
    max-width: 470px;
    max-height: 450px;
    border-radius: 8px;
    object-fit: cover;
  }
}

@media screen and (max-width: 540px) {
  .loader {
    p {
      font-size: 25px;
    }
  }
  .images {
    margin-top: 50px;
    grid-template-columns: repeat(1, 1fr);
    gap: 17px;
    margin-bottom: 130px;
    padding: 0px 20px;
  }
}
</style>
