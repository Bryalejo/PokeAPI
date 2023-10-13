<template>
  <div class="">
    <h1 class="text-5xl text-center">Carrusel de Pokémon</h1>
    <swiper
      :slidesPerView="3"
      :spaceBetween="30"
      :pagination="{
        clickable: true,
      }"
      class="mySwiper h-full my-20 mx-20 text-xl"
    >
      <swiper-slide v-for="pokemon in pokemonList" :key="pokemon.name">
        <img src="/imgs/pokemon1.png" alt="" class="py-5 w-full" />
        <div class="ml-5">
          Nombre: <span class="">{{ pokemon.name }}</span> <br />
          <button
            @click="showPokemonDetails(pokemon)"
            class="bg-red-500 shadow-lg transition ease-in-out delay-150 hover:-translate-y-1 hover:scale-110 hover:bg-yellow-500 duration-300 text-white shadow-red-500/50 rounded-md px-2 outline outline-offset-2 outline-red-600 my-5"
            type="button"
          >
            Detalles
          </button>
        </div>
      </swiper-slide>
    </swiper>

    <Modal v-if="isShowModal" @close="closeModal">
      <template #header>
        <div class="flex items-center text-4xl text-white">
          {{ selectedPokemon.name }}
        </div>
      </template>
      <template #body>
        <!-- Use the PokemonDetails component here -->
        <div class="text-white font-bold text-xl">
          <ul>
            <li>
              Indice de felicidad:
              {{ selectedPokemon.additionalInfo.baseHappiness }}
            </li>
            <li>
              Tasa de Captura: {{ selectedPokemon.additionalInfo.captureRate }}
            </li>
            <li
              v-if="
                selectedPokemon?.additionalInfo?.
                color?.name
              "
            >
              Color: {{ selectedPokemon.additionalInfo.color.name }}
            </li>
            <li v-else>Color: No se encontró información de color.</li>
            <li
              v-if="
                selectedPokemon?.additionalInfo?.eggGroups?.[0]?.name" 
            >
              Especie:{{ selectedPokemon.additionalInfo.eggGroups[0].name }}
             
            </li>
            <li v-else>Tipo: null</li>
            <li
              v-if="
                selectedPokemon?.additionalInfo?.eggGroups?.[1]?.name"
              
            >
              
              Tipo: {{ selectedPokemon.additionalInfo.eggGroups[1].name  }}
            </li>
            <li v-else>Tipo: null</li>

            <li
              v-if="
                selectedPokemon?.additionalInfo?.evolvesFromSpecies?.name
              "
            >
              Pokemon pre-evolucion:
              {{ selectedPokemon.additionalInfo.evolvesFromSpecies.name }}
            </li>
            <li v-else>Evolucion: null</li>
          </ul>
        </div>
      </template>
    </Modal>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import axios from "axios";
import { Swiper, SwiperSlide } from "swiper/vue";
import { Pagination } from "swiper/modules";
import { Modal } from "flowbite-vue";

import "swiper/css";
import "swiper/css/pagination";

export default {
  components: {
    Swiper,
    SwiperSlide,
    Modal,
  },
  setup() {
    const pokemonList = ref([]);
    const isShowModal = ref(false);
    const selectedPokemon = ref(null);

    const fetchPokemonList = () => {
      axios
        .get("https://pokeapi.co/api/v2/pokemon-species")
        .then(async (response) => {
          const pokemonSpeciesList = response.data.results;

          // Realizar solicitudes adicionales para obtener información adicional
          const promises = pokemonSpeciesList.map(async (pokemon) => {
            const response = await axios.get(pokemon.url);
            const additionalInfo = {
              baseHappiness: response.data.base_happiness,
              captureRate: response.data.capture_rate,
              color: response.data.color,
              eggGroups: response.data.egg_groups,
              evolvesFromSpecies: response.data.evolves_from_species,
            };
            return {
              ...pokemon,
              additionalInfo,
            };
          });

          // Esperar a que se completen todas las solicitudes adicionales
          const pokemonListWithInfo = await Promise.all(promises);
          pokemonList.value = pokemonListWithInfo;
        })
        .catch((error) => {
          console.error(
            "Error al cargar la lista de Pokémon desde la API",
            error
          );
        });
    };

    const closeModal = () => {
      isShowModal.value = false;
      selectedPokemon.value = null;
    };

    const showPokemonDetails = (pokemon) => {
      selectedPokemon.value = pokemon;
      isShowModal.value = true;
    };

    onMounted(() => {
      fetchPokemonList();
    });

    return {
      modules: [Pagination],

      pokemonList,
      isShowModal,
      selectedPokemon,
      closeModal,
      showPokemonDetails,
    };
  },
};
</script>
