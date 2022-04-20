<template>
  <q-page class="mx-auto">
    <q-page-container class="flex flex-center mt-10 mb-2">
      <div class="text-h3 text-center mr-3">Trending movies</div>
      <q-btn-toggle
        v-model="state.period"
        color="blue-grey-10"
        toggle-color="white"
        toggle-text-color="black"
        rounded
        :options="[
          {label: 'weekly', value: 'week'},
          {label: 'Diary', value: 'day'},
        ]"
      />
      <!-- <q-btn-dropdown outline :label="state.period" dropdown-icon="change_history">
        <q-list>
          <q-item clickable v-close-popup @click="state.period = 'week'">
            <q-item-section>
              <q-item-label>Weekly</q-item-label>
            </q-item-section>
          </q-item>

          <q-item clickable v-close-popup @click="state.period = 'day'">
            <q-item-section>
              <q-item-label>Diary</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>-->
    </q-page-container>
    <div class="flex flex-center justify-center">
      <q-intersection
        transition="scale"
        once
        class="q-pa-md row items-start q-gutter-md"
        v-for="movie in responseTrendsMovies"
        :key="movie.id"
        @click="()=>{
          state.dialog.show = true
          state.dialog.data = movie
        }"
      >
        <q-card class="movie">
          <div class="box">
            <q-img
              spinner-color="white"
              :src="`https://image.tmdb.org/t/p/w300${movie.poster_path}`"
              class="thumb"
            />

            <q-card class="details">
              <q-card-section>
                <div class="text-h5">{{ movie.title }}</div>
                <div class="flex justify-between q-mt-sm">
                  <span class="movie-note text-subtitle2 flex flex-center">
                    <q-icon :name="movie.vote_average ? 'star' : 'star_border'"></q-icon>
                    <span class="ml-1">{{ movie.vote_average }}</span>
                  </span>
                  <span class="movie-year">{{ format(new Date(movie.release_date), 'yyyy') }}</span>
                </div>

                <!-- <q-rating
                max="5"
                size="3em"
                color="yellow"
                icon="star_border"
                icon-selected="star"
                icon-half="star_half"
                no-dimming
                disable
                />-->
              </q-card-section>
            </q-card>
          </div>
        </q-card>
      </q-intersection>
    </div>
    <q-dialog v-model="state.dialog.show">
      <q-card class="w-[600px] h-auto">
        <q-img
          spinner-color="white"
          :src="`https://image.tmdb.org/t/p/w500${state.dialog.data.backdrop_path}`"
          class="w-full"
        />

        <q-card>
          <q-card-section>
            <div class="flex justify-between q-mt-sm">
              <div class="text-h5">{{ state.dialog.data.title }}</div>
              <span class="movie-note text-subtitle2 flex flex-center">
                <q-icon :name="state.dialog.data.vote_average ? 'star' : 'star_border'"></q-icon>
                <span class="ml-1">{{ state.dialog.data.vote_average }}</span>
              </span>
            </div>
            <div class="text-md my-2">{{ state.dialog.data.overview }}</div>
            <div class="flex gap-2 mt-2">
              <span class="movie-note text-subtitle2 flex flex-center bg-gray-700">
                <q-icon name="language"></q-icon>
                <span class="uppercase ml-1">{{ state.dialog.data.original_language }}</span>
              </span>

              <span class="movie-note text-subtitle2 flex flex-center bg-gray-700">
                <q-icon name="movie"></q-icon>
                <span
                  class="ml-1"
                >{{ format(new Date(state.dialog.data.release_date), 'yyyy/MM/dd') }}</span>
              </span>
            </div>
          </q-card-section>
        </q-card>
      </q-card>
    </q-dialog>
    <div class="flex-center flex loading" v-intersection="onIntersection">
      <q-spinner-bars color="white" size="4em" />
    </div>
  </q-page>
</template>

<script>

import { defineComponent, ref, onBeforeMount, watch, reactive } from 'vue'
import { api } from 'boot/axios'
import { useQuasar } from 'quasar'
import { format } from 'date-fns'
export default defineComponent({
  name: 'IndexPage',
  setup () {
    const key = 'fb8e26c908ec4f585c6be0cc6e04c393'
    const $q = useQuasar()
    const responseTrendsMovies = ref([])
    const period = reactive('week')
    const state = reactive({
      page: 1,
      period: 'week',
      dialog: {
        show: false,
        movieData: {}
      }
    })

    const getData = async () => {
      await api.get(`/trending/movie/${state.period}?api_key=${key}&page=${state.page}&per_page=10`)
        .then((response) => {
          if (response.data) {
            console.log(response.data)
            responseTrendsMovies.value = [
              ...responseTrendsMovies.value,
              ...response.data.results
            ]
          }
        })
        .catch((err) => {
          let msg
          if (err.response) {
            msg = err.response.data.message
          } else {
            msg = 'Erro na conexão!'
          }
          $q.notify({
            color: 'negative',
            position: 'top',
            message: msg,
            icon: 'report_problem'
          })
        })
    }
    watch(() => state.page, () => {
      getData()
    })
    watch(() => state.period, () => {
      responseTrendsMovies.value = []
      state.page = 1
    })
    onBeforeMount(async () => {
      await getData()
    })
    return {
      responseTrendsMovies,
      period,
      state,
      getData,
      format,
      onIntersection (entry) {
        setTimeout(() => {
          state.page++
        }, 4 * 1000)
      }
    }
  }

})
</script>
<style scoped>
.page {
  padding-bottom: 10rem;
}
.movies-title {
  margin: 5rem 0 1rem 3rem;
}
.movie {
  cursor: pointer;
  width: 300px;
  height: 450px;
  position: relative;
}
.thumb {
  width: 300px;
  border-radius: 5px;
}
.movie .box {
  height: 450px;
  position: absolute;
  border-radius: 5px;
}
.details {
  visibility: hidden;
  margin-top: -0.5rem;
  width: 300px;
  height: 0;
  position: absolute;
  z-index: 50;
}

.movie:hover .details {
  visibility: visible;
  width: 330px;
  height: auto;
  display: block;
  border-radius: 0 0 5px 5px;
  transition: all 0.4s ease;
}
.movie:hover .box {
  position: absolute;
  width: 330px;
  height: 480px;
  margin-left: -15px;
  margin-top: -15px;
  transition: all 0.4s ease;
}
.movie:hover .thumb {
  border-radius: 5px 5px 0 0;
  width: 330px;
  transition: all 0.4s ease;
}
.movie-note {
  background: rgb(19, 19, 19);
  border-radius: 5px;
  padding: 0.4rem;
}
.movie-year {
  font-size: 1.1rem;
  padding: 0.3rem;
}
.loading {
  margin-top: 5rem;
  width: 100%;
}
</style>
