<script setup lang="ts">
import HeaderComponent from '@/components/HeaderComponent.vue'
import { ref } from 'vue'

const favoritePhotos = ref()

if (localStorage.getItem('favorites')) {
  favoritePhotos.value = JSON.parse(localStorage.getItem('favorites')!)
} else {
  favoritePhotos.value = []
}
</script>

<template>
  <HeaderComponent />
  <div class="container">
    <h1 class="favorites-title">
      {{ favoritePhotos.length === 0 ? 'Избранного пока нет' : 'Избранное' }}
    </h1>
    <div class="images">
      <router-link v-for="photo in favoritePhotos" :key="photo?.id" :to="'/details/' + photo?.id">
        <img class="image" :src="photo?.urls?.regular" alt="" />
      </router-link>
    </div>
  </div>
</template>

<style scoped lang="scss">
.favorites-title {
  font-weight: 700;
  font-size: 60px;
  text-align: center;
  color: #000000;
  margin-top: 50px;
  margin-bottom: 70px;
}
.images {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 30px;
  margin-bottom: 200px;
  img {
    width: 100%;
    height: 100%;
    max-width: 470px;
    max-height: 450px;
    border-radius: 8px;
    object-fit: cover;
  }
}

@media screen and (max-width: 540px) {
  .favorites-title {
    font-size: 30px;
    margin-top: 30px;
    margin-bottom: 50px;
  }
  .images {
    grid-template-columns: repeat(1, 1fr);
    gap: 17px;
    margin-bottom: 130px;
    padding: 0px 20px;
  }
}
</style>
