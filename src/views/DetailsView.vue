<script setup lang="ts">
import HeaderComponent from '@/components/HeaderComponent.vue'
import { ref, computed } from 'vue'
import { useRoute } from 'vue-router'
import axios from 'axios'

const loading = ref(true)
const route = useRoute()
const id = ref(route.params.id)
const photo = ref()

axios
  .get(`https://api.unsplash.com/photos/${id.value}`, {
    headers: {
      Authorization: `Client-ID ${import.meta.env.VITE_ACCESS_KEY}`
    }
  })
  .then((response) => {
    photo.value = response.data
    loading.value = false
  })
  .catch((error) => {
    console.error(error)
  })

const downloadImage = async (imageUrl: string) => {
  const url = window.URL.createObjectURL(new Blob([imageUrl]))
  const link = document.createElement('a')
  link.href = url
  link.setAttribute('download', 'unsplash-image.jpg')
  document.body.appendChild(link)
  link.click()
}

const favorites = ref()

if (localStorage.getItem('favorites')) {
  favorites.value = JSON.parse(localStorage.getItem('favorites')!)
} else {
  favorites.value = []
}

const addFavorite = (image: any) => {
  const index = favorites.value.findIndex((fav: any) => fav.id === image.id)
  if (index === -1) {
    favorites.value.push(image)
    localStorage.setItem('favorites', JSON.stringify(favorites.value))
  } else {
    removeFavorite(image)
  }
}

const removeFavorite = (image: any) => {
  const index = favorites.value.findIndex((fav: any) => fav.id === image.id)
  if (index > -1) {
    favorites.value.splice(index, 1)
    localStorage.setItem('favorites', JSON.stringify(favorites.value))
  }
}

const isFavorite = computed(() => {
  return favorites.value.some((fav: any) => fav.id === photo.value?.id)
})
</script>

<template>
  <HeaderComponent />
  <div
    v-if="photo || loading"
    class="full-bg"
    :style="`background: url(${photo?.urls.regular}) no-repeat center center fixed; background-size: cover; -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;`"
  >
    <div class="container detail">
      <div class="loader" v-if="loading">
        <p>Загрузка...</p>
      </div>
      <div v-if="!loading" class="detail__header">
        <div class="detail__header-author">
          <img :src="photo.user.profile_image.large" alt="" />
          <div class="detail__header-author-name">
            <p>{{ photo.user.first_name }} {{ photo.user.last_name }}</p>
            <p>@{{ photo.user.username }}</p>
          </div>
        </div>
        <div class="detail__header-btns">
          <button class="detail__header-btns-fav" @click="addFavorite(photo)">
            <svg
              v-if="!isFavorite"
              width="25"
              height="23"
              viewBox="0 0 25 23"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M18.652 0.118408C15.9926 0.118408 13.7961 2.58857 12.9078 3.75439C12.0195 2.58857 9.82303 0.118408 7.16359 0.118408C3.72082 0.118408 0.919922 3.28368 0.919922 7.17379C0.919922 9.29616 1.76235 11.2741 3.23691 12.6278C3.25835 12.6648 3.28472 12.699 3.31544 12.7297L12.5551 21.9503C12.6527 22.0474 12.78 22.0962 12.9078 22.0962C13.0356 22.0962 13.1634 22.0474 13.261 21.9499L22.808 12.4063L22.9065 12.3107C22.9846 12.237 23.0616 12.1624 23.1479 12.0668C23.184 12.0312 23.2138 11.9912 23.2367 11.9478C24.3074 10.6366 24.8957 8.94542 24.8957 7.17379C24.8957 3.28368 22.0948 0.118408 18.652 0.118408ZM22.4055 11.3858C22.3919 11.402 22.3792 11.419 22.368 11.4366C22.3211 11.4902 22.2699 11.5375 22.2192 11.5858L12.9073 20.8909L4.13544 12.1365C4.10715 12.0809 4.06763 12.0302 4.01935 11.9878C2.68425 10.8219 1.9189 9.06736 1.9189 7.17379C1.9189 3.83437 4.27152 1.11743 7.16359 1.11743C10.023 1.11743 12.4634 4.84706 12.4878 4.88462C12.6722 5.16947 13.1434 5.16947 13.3278 4.88462C13.3522 4.84706 15.7926 1.11743 18.652 1.11743C21.5441 1.11743 23.8967 3.83442 23.8967 7.17379C23.8967 8.74739 23.3669 10.2434 22.4055 11.3858Z"
                fill="black"
              />
            </svg>
            <svg
              v-else
              width="24"
              height="22"
              viewBox="0 0 24 22"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
              xmlns:xlink="http://www.w3.org/1999/xlink"
            >
              <rect y="1" width="23" height="20" fill="url(#pattern0)" />
              <path
                d="M17.7321 0C15.0727 0 12.8762 2.47016 11.9879 3.63598C11.0996 2.47016 8.90311 0 6.24367 0C2.8009 0 0 3.16527 0 7.05538C0 9.17775 0.84243 11.1557 2.31699 12.5094C2.33843 12.5464 2.3648 12.5806 2.39552 12.6113L11.6352 21.8319C11.7328 21.929 11.8601 21.9778 11.9879 21.9778C12.1157 21.9778 12.2435 21.929 12.3411 21.8315L21.888 12.2879L21.9866 12.1923C22.0646 12.1186 22.1417 12.044 22.228 11.9484C22.2641 11.9128 22.2939 11.8728 22.3168 11.8294C23.3875 10.5182 23.9758 8.82701 23.9758 7.05538C23.9758 3.16527 21.1749 0 17.7321 0ZM21.4856 11.2674C21.4719 11.2835 21.4592 11.3006 21.448 11.3182C21.4012 11.3718 21.35 11.4191 21.2993 11.4674L11.9874 20.7725L3.21551 12.0181C3.18723 11.9625 3.14771 11.9118 3.09943 11.8694C1.76433 10.7035 0.998974 8.94895 0.998974 7.05538C0.998974 3.71596 3.3516 0.999021 6.24367 0.999021C9.10311 0.999021 11.5435 4.72866 11.5679 4.76621C11.7523 5.05106 12.2235 5.05106 12.4079 4.76621C12.4323 4.72866 14.8727 0.999021 17.7321 0.999021C20.6242 0.999021 22.9768 3.71601 22.9768 7.05538C22.9768 8.62898 22.447 10.125 21.4856 11.2674Z"
                fill="black"
              />
              <defs>
                <pattern id="pattern0" patternContentUnits="objectBoundingBox" width="1" height="1">
                  <use
                    xlink:href="#image0_205_6"
                    transform="matrix(0.00195312 0 0 0.00224609 0 -0.003125)"
                  />
                </pattern>
                <image
                  id="image0_205_6"
                  width="512"
                  height="448"
                  xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAgAAAAHABAMAAAAmVs3gAAAAAXNSR0IArs4c6QAAAARnQU1BAACxjwv8YQUAAAASUExURQAAAOsYIe0aI+0aI+0aI+0bJOk5H/0AAAAFdFJOUwAvY5nQVvsvJwAACbhJREFUeAHlwbd63cYahtEXgf1Ih+hhyuhh0+hB2dMT4bv/WzmPRAWGHYCNPP9aHNOnhy+Pj//999/j48Mfjk1Enx4eHh4f/354+OMTq/r8xeutfx8cq/r05R+v1/rHh0+s4vM/Oqn/27GSz49ep/SPf7C0/3md92/O8qIvXuf1fzsW9D+vy/7NWVb0Rdd8dSzks9d1Xx0L+qIhvjoWEP2jYf5iKZ+9BvqL2X32GqpzLCH6U8N1jnn9qTH+Yn6x1yh/MaOo0jhfmdsXjfWV2UReY3WOOUWVxusc84i9xutz5hNVukXnmEPsdZOcucRet+kd08W6Vck8Yq9b9TlTxV43K5lDrAn6nGkirwlKpos1Se+YpNIkOVNFXtN0jgkKTZQzTeQ1Vcft7jVV75ik0nQtt0o0Xe+YoNAcnrhN5DWDltvdax45N6k0iyduFWsmveMG95pJyW0ir7m0jBdrNjk3KTSfJ0arNJuOWySaU85I95pRw3iRZtUxTqxZlYxWaF4No1SaVe8YKdHcckZINbOWcSKvuXUMF2l2JaNkmt8TgxWaXc8YsZaQM1CsBTSMUGkJHQNVWkLOYImWUTJIokW0DOa1jN4xhNcySgZKtZSGAe61kJ6BvBaTc1WkxTwxSKrltFyVaTmOIbwWVHJFpAU1DJBqST1XZFqS4zqvRdVcFGtRDVclWlbvuKTQshzXVFpYwwWxFtZwRazFOc4rtDTHZYUW13JWrMU1XBRpBTnnFFqe45I7raDljFgrqLnEaw05pxVaQc8FiVbRclKsVZScV2gdJadkWkXHWZFW0nFCpJXknJNqLSUfZVpJwzmF1tLxQaTVOE6LtJ6S9zKtpua0VOvpeSfSejpOq7SimrcyrSjnlEhr6nkj0poaTkm1qprXUq2p55RCq+p5zWtVJSdoZTW/pVpXw0eJVtbzm9fK+CjT2mp+SrS2kg+81tbzU6W1NbwXaX01LxKtj/dSra93fFdpfSXvFNrAE9/E2kDDO15bcACFNtDzVqxNNECsTeS8kWobDgpt4pk3Mm2jIdY2Ot7w2ojLtBHHK5G20morNa8ksqfhlTvZ0/NKIYNyfvMyqOaXSBa1/JLIop5f7mRSzk+FTKr5qZJJDT/Jpo4fYhnleJHIqJIXdzLqmReZjGp5UcmonhdeVjm+k1kl38Qy65lvEpnV8E0qszq+uZNdfJPJLgdQyK4SoJJdzwBedjUAMqwFIhnWA7EsAxJZ5iCRZSWksqyGO1n2DHeyrIFMlrVQyLIOClnWQyHToJJpjkqm5XiZVuJlWomXaTWy7RnZ1iDbGmRbi2xrkW0dsq1DtnXIth7Z1iPjkHF42YaXbXjZRiXbqGQblWyjkG0Uso1MtpHJtJ47mdZzJ9N6UpnWkcq0jkSmdcQyrSWSaS2RTGtApj2Dl2XPUMmyGgpZVkImy0rIZFkOd7LMQSLLgFiG9UAkwzoAGdYCeNnVAFSy6xmgkF01QCa7coBUdjmARHbxTSyzOr6TWS3feVnV8F0hq2q+y2RVyXd3ssrxXSKreBHJqI4fZFTDD5VseuaHQjaV/HAnmxw/JDKp56dIJrX8IpOe+aWSRSW/ZLLI8Usqg3p+i2VQyysy6JlXKtmT80ome3jtTua0vBbLnGfekDklb1SyhrcyGdPxVipjnnkrljE578gY3itkSst7dzKl5r1Epjg+kCUdH1UypOGjTIaUfJTIEE6IZEfLKZXMKDklkxmOUxJZ0XJSJCtqTqtkhOO0O9nQckYiG2rOkQ2OcwpZ0HFWKgtqzoplgeM8r/C1XJApfDUXJAqf4xIFr+WiQqEruShV4HouixS4hisqhS3nikxB67gmVtCeucorZI6rMgWs5bpEASsZQOHqGaJQsBqGSBSsnEEUqo5hCgWqZphEgWIohalhqEJByhkqUYg6hlOIaobLFJ6eERKFp2EMr+DkjJEpNC2jxApNyTheYekZKVNYnhkpVlgcY1UKScNoqUKSM1qkgHTcoFA4Sm6QKBg9N/EKxTM3yRQKx01iBaLhRpXCkHOjVEFouZmCUHKzTAHouF2sANRMUOnweqZIdXgNk+jwHJNkOriGaWIdnGOiQofWMlWiQ8uZzOvAOqbLdGAl00U6ro45FDqsmjnEOqqeeVQ6qJp5pDqmnrl4HVLDXDIdkmMukY6oYT6ZDsgxn1jH0zCnQoeTM6dER9Myr0oHkzOvVMfSMjevQ8mZW6YjaZldpCMpmV+m4+hYQKTjKFlCoaPoWESsoyhZRqFj6FhIomMoWUqlI+hYTKIjKFlOpf3rWFCq/atZktfe9Swq1d7VLMtr33oWlmrfapbmtWc9i8u0ZzWLi7RjPSvItF81K4i0Wz2ryLRXNauItFM9K8m0TzUribRLPavJtEclq4m0Qx0ryrQ/JSuKtDsdq8q0NyWrirz2pWNl99qXkpVFXnvSsbpUe1KyPq/96NhAqv0o2YLXXnRsItVe5GzDax9aNpJoH3K2UmkPWjaTaA9ytlNpey0birW9nC0V2lrLpmJtzbGtQttq2FikbTm2lmlLDZuLtCXH9jJtp2EHIm3HsQf32soT++C1jZ6dSLWNmr2otIWe3Ui0hZr9qLS+nh2Jtb6SPSm0to5dibW2kn3JtK6OnYm0rpK9ybSmlv3xWlHO/qRaT8seea0mZ48SraVlnyqtJGefYq2jYa8KrcKxV7HW0LBfmVbg2K9Iy2vYs3stzrFrXgt7Yt9SLat37FylRdXsXaIl9exfpQXV7F+s5fQcQaHF1BxBpKV0HEOmhZQcQ6RldBzFvRZRchheC+g4jlQLKDmQSrPrOJJEs8s5lEIzazmWWDPLOZhCs2o5mkizyjmcTDNqOZ5IM8o5oHvNpuWQvOaSc0ipZtJyUJXmkXNQiWbRcliV5pBzWIlm0HJghabLObBYk7UcWqGpcg4t1kQNB5dpmpyDizRJw+FlmsJxeJEmaAhApts5AhDpZg1BuNetHGHwuk1DIFLdxhEKr1s0BCPVLRzh8BqvISCJxnOEpNJYDUFJNJYjLJXGaQhMonEcoak0RkNwYo3hCE+h4RoCFGs4R4gKDdUQpFhDOcJUaJiGQEUaxhGqTEM0BCvSEI5wZbquIWCRrnOELNM1DUGLdI0jbPe6rCFwkddFjtCluqQhfF4XOMKX6rwnLPA6p3dYkOqcJ2yodFrvsCHRaTVWVDqlx4xEp9TYUemjHkNifVRjSaH3ekyJ9V6NLYXe6jEm1ls11hR6rcecSK/V2JPptx6DIv1WY1Gmn3pMivRTjU2ZXvQYFelFjVWZvukxK/KSVGNXKqnHMi/VWJaqxzZfY1vM1v4PkuuJQgN1mJMAAAAASUVORK5CYII="
                />
              </defs>
            </svg>
          </button>
          <button class="detail__header-btns-download" @click="downloadImage(photo.links.download)">
            <svg
              width="26"
              height="23"
              viewBox="0 0 26 23"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M25.381 10.7776C25.0395 10.7787 24.7629 11.055 24.7619 11.3966V19.209C24.759 20.2333 23.9291 21.0631 22.9048 21.0662H3.09524C2.07091 21.0631 1.241 20.2333 1.2381 19.209V11.3966C1.2381 11.0548 0.961071 10.7776 0.619048 10.7776C0.277024 10.7776 0 11.0548 0 11.3966V19.209C0.00077381 20.9182 1.38609 22.3035 3.09524 22.3043H22.9048C24.6139 22.3035 25.9992 20.9182 26 19.209V11.3966C25.999 11.055 25.7224 10.7787 25.381 10.7776Z"
                fill="black"
              />
              <path
                d="M8.46874 14.3124L12.4059 17.6862C12.4094 17.6862 12.4121 17.6891 12.4121 17.6924L12.4306 17.711C12.4368 17.7171 12.4492 17.7233 12.4554 17.7295C12.4614 17.7373 12.4703 17.7417 12.4802 17.7419C12.4864 17.7481 12.4987 17.7543 12.5049 17.7605C12.5111 17.7667 12.5235 17.7667 12.5297 17.7729C12.5359 17.779 12.5483 17.779 12.5606 17.7852C12.5668 17.7914 12.5792 17.7914 12.5854 17.7976C12.5978 17.8038 12.604 17.8038 12.6164 17.81C12.6225 17.81 12.6349 17.8162 12.6411 17.8162C12.6535 17.8162 12.6597 17.8224 12.6721 17.8224C12.6783 17.8224 12.6906 17.8286 12.6968 17.8286C12.7092 17.8286 12.7216 17.8348 12.734 17.8348H12.8949C12.9073 17.8348 12.9197 17.8286 12.9321 17.8286C12.9383 17.8286 12.9506 17.8224 12.9568 17.8224C12.9692 17.8224 12.9754 17.8162 12.9878 17.8162C12.994 17.8162 13.0064 17.81 13.0125 17.81C13.0249 17.8038 13.0311 17.8038 13.0435 17.7976C13.0497 17.7914 13.0621 17.7914 13.0683 17.7852C13.0806 17.779 13.0868 17.779 13.0992 17.7729C13.1054 17.7667 13.1178 17.7667 13.124 17.7605C13.1302 17.7543 13.1425 17.7481 13.1487 17.7419C13.1549 17.7357 13.1611 17.7357 13.1735 17.7295C13.1797 17.7233 13.1921 17.7171 13.1983 17.711L13.2168 17.6924C13.2203 17.6924 13.223 17.6895 13.223 17.6862L17.1602 14.3124C17.4202 14.0901 17.4505 13.6993 17.2283 13.4395C17.0062 13.1797 16.6154 13.1492 16.3554 13.3714L13.4335 15.8724V0.619048C13.4335 0.277024 13.1565 0 12.8145 0C12.4724 0 12.1954 0.277024 12.1954 0.619048V15.8724L9.2735 13.3714C9.0135 13.1492 8.62272 13.1797 8.40064 13.4395C8.17836 13.6993 8.20874 14.0901 8.46874 14.3124Z"
                fill="black"
              />
            </svg>
            <p>Downloand</p>
          </button>
        </div>
      </div>
      <img v-if="!loading" class="detail__main-img" :src="photo.urls.regular" alt="" />
    </div>
  </div>
</template>

<style scoped lang="scss">
.full-bg {
  width: 100%;
  height: 100%;
  padding-bottom: 110px;
  .detail {
    display: flex;
    flex-direction: column;
    gap: 40px;
    padding-top: 50px;
    justify-content: center;
    .loader {
      display: flex;
      justify-content: center;
      margin-top: 0px;
      p {
        font-weight: 700;
        font-size: 30px;
        color: #000000;
      }
    }
    h1 {
      font-weight: 700;
      font-size: 60px;
      color: #ffffff;
    }
    .detail__header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      .detail__header-author {
        display: flex;
        align-items: flex-end;
        gap: 10px;
        img {
          width: 55px;
          height: 55px;
          border: 1px solid #ffffff;
          border-radius: 8px;
          object-fit: cover;
        }
        .detail__header-author-name {
          display: flex;
          flex-direction: column;
          white-space: nowrap;
          :nth-child(1) {
            font-weight: 400;
            font-size: 30px;
            color: #f2f2f2;
            line-height: -500%;
          }
          :nth-child(2) {
            font-weight: 400;
            font-size: 18px;
            color: #f2f2f2;
            margin-top: -10px;
          }
        }
      }
      .detail__header-btns {
        display: flex;
        align-items: center;
        gap: 20px;
        .detail__header-btns-fav {
          padding: 14px 17px 9px 15px;
          background: #ffffff;
          box-shadow: 0px 0px 4px rgba(0, 0, 0, 0.25);
          border-radius: 8px;
          border: none;
          transition: all 0.3s ease;
        }
        .detail__header-btns-fav:hover {
          background: #fff200;
        }
        .detail__header-btns-download {
          display: flex;
          align-items: center;
          gap: 16px;
          padding: 13px 35px 13px 23px;
          background: #fff200;
          border: 1px solid #fff200;
          box-shadow: 0px 0px 4px rgba(0, 0, 0, 0.25);
          border-radius: 8px;
          transition: all 0.3s ease;
          p {
            font-weight: 400;
            font-size: 20px;
            color: #000000;
          }
        }
        .detail__header-btns-download:hover {
          background: #f99d00;
          border: 1px solid #f99d00;
        }
      }
    }
    .detail__main-img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      max-height: 700px;
      box-shadow: 0px 4px 50px rgba(0, 0, 0, 0.5);
      border-radius: 8px;
    }
  }
}

@media screen and (max-width: 540px) {
  .loader {
    p {
      font-size: 25px !important;
    }
  }
  .full-bg {
    background: white !important;
  }
  .detail {
    padding: 0px 20px;
  }
  .detail__header-btns-fav {
    padding: 12px 12px 8px 12px !important;
  }
  .detail__header-btns-download {
    padding: 10px !important;
    p {
      display: none;
    }
  }
  .detail__header-author {
    align-items: flex-start !important;
  }
  .detail__header-author-name {
    :nth-child(1) {
      font-size: 18px !important;
      color: #000000 !important;
    }
    :nth-child(2) {
      font-size: 14px !important;
      color: #bdbdbd !important;
    }
  }
}
</style>
