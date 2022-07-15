<script setup>
  import {ref, computed, onMounted } from 'vue'

  const query = ref('')
  const my_anime = ref([])
  const search_results = ref([])

  const my_anime_asc = computed(() =>{
    return my_anime.value.sort((a , b) => {
      return a.title.localeCompare(b.title)
    })
  })

  const searchAnime = () =>{
    const url = `https://api.jikan.moe/v4/anime?q=${query.value}`
    fetch(url)
      .then(res => res.json())
      .then(res => {
        search_results.value = res.data
      })
  }


  const handleInput = e =>{
    if(!e.target.value){
      search_results.value = []
    }
  }

  const addAnime = anime => {
    search_results.value = []
    query.value =''

    my_anime.value.push({
      id: anime.mal_id,
      title: anime.title,
      image: anime.images.jpg.image_url,
      total_episodes: anime.episodes,
      watched_episodes: 0,
    })

    localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
  }


  const increaseWatch = anime => {
    anime.watched_episodes++
      localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
  }

    const decreaseWatch = anime => {
    anime.watched_episodes--
      localStorage.setItem('my_anime', JSON.stringify(my_anime.value))
  }

  onMounted(() => {
    my_anime.value = JSON.parse(localStorage.getItem('my_anime')) || []
  })



</script>

<template>
  <main>
    <h1>Anime Tracker</h1>

    <form @submit.prevent="searchAnime">
      <input type="text" placeholder="Search for an anime.." 
              v-model="query" @input="handleInput" />

      <button type="submit" class="button">Search</button>

    </form>

    <div class="results" v-if="search_results.length > 0">

      <div class="result" v-for="anime in search_results">

        <img :src="anime.images.jpg.image_url" />
        <div class="details">

          <h3> {{ anime.title }} </h3>
          <p :title ="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...</p>

          <span class="flex-1"></span>
          <button class="button" @click="addAnime(anime)"> Add to my anime </button>
        </div>
      </div>
    </div>

          <div class="myanime" v-if="my_anime.length > 0">
            <h2>My Anime</h2>

            <div v-for="anime in my_anime_asc" class="anime">
              <img :src="anime.image" />
              <h3> {{ anime.title }} </h3>
              <div class="flex-1"></div>
              <span class="episodes">
                {{ anime.watched_episodes }} / {{ anime.total_episodes }}   </span>

                <button class="button" v-if="anime.total_episodes !== anime.watched_episodes"
                @click="increaseWatch(anime)">   +   </button>

                <button class="button" v-if="anime.watched_episodes > 0"
                @click="decreaseWatch(anime)">   -   </button>
   
            </div>
          </div>
  </main>


</template>


