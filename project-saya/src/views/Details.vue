<script setup>
import { ref, onMounted, computed } from "vue";
import { useRoute } from "vue-router";
import icon from "@/components/icon.vue";
// import Navbar from "@/components/Navbar.vue";
import icon_side from "@/components/icon_side.vue";
// import LoadingSpiner from './components/LoadingSpiner.vue';

const route = useRoute();
const pokemon = ref(null);
const isLoading = ref(true);
setTimeout(() => {
  isLoading.value = false;
}, 3000);

const errorMessage = ref("");
const activeTab = ref("about");
const isCatching = ref(false);
const caughtMessage = ref(null);
let capturedPokemon = JSON.parse(localStorage.getItem("pokemonList")) || [];

const typeColors = {
  grass: "bg-green-500",
  fire: "bg-red-500",
  water: "bg-blue-500",
  electric: "bg-yellow-400",
  psychic: "bg-purple-500",
  ice: "bg-cyan-300",
  dragon: "bg-indigo-600",
  dark: "bg-gray-800",
  fairy: "bg-pink-300",
  normal: "bg-gray-400",
  fighting: "bg-orange-600",
  flying: "bg-sky-400",
  poison: "bg-purple-400",
  ground: "bg-yellow-700",
  rock: "bg-yellow-800",
  bug: "bg-lime-500",
  ghost: "bg-indigo-400",
  steel: "bg-gray-500",
};

const backgroundColor = computed(() => {
  if (pokemon.value && pokemon.value.types.length > 0) {
    const primaryType = pokemon.value.types[0];
    return typeColors[primaryType] || "bg-gray-200";
  }
  return "bg-gray-200";
});

const fetchPokemonDetail = async (id) => {
  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${id}`);
    if (!res.ok) throw new Error("Gagal fetching data");
    const data = await res.json();
    const totalStats = data.stats.reduce(
      (sum, stat) => sum + stat.base_stat,
      0
    );
    const averageStats = totalStats / data.stats.length;

    pokemon.value = {
      id: data.id,
      name: data.name.charAt(0).toUpperCase() + data.name.slice(1),
      image:
        data.sprites.other["official-artwork"].front_default || "/default.png",
      types: data.types.map((t) => t.type.name),
      height: `${data.height / 10} (M)`,
      weight: `${data.weight / 10} (KG)`,
      abilities: data.abilities.map((a) => a.ability.name),
      experience: data.base_experience,
      moves: data.moves,
      stats: data.stats,
      averageStats: averageStats.toFixed(2),
    };
  } catch (error) {
    errorMessage.value = "Error fetching Pokémon data.";
  } finally {
    isLoading.value = false;
  }
};

const catchPokemon = () => {
  if (!pokemon.value || isCatching.value) return;

  isCatching.value = true;
  caughtMessage.value = null;

  setTimeout(() => {
    const success = Math.random() > 0.5;
    if (success) {
      const newPokemon = {
        id: pokemon.value.id,
        name: pokemon.value.name,
        types: pokemon.value.types,
        image: pokemon.value.image,
        uniqueId: Date.now(),
      };
      capturedPokemon.push(newPokemon);
      localStorage.setItem("pokemonList", JSON.stringify(capturedPokemon));

      caughtMessage.value = {
        success: true,
        pokemon: newPokemon,
      };
    } else {
      caughtMessage.value = {
        success: false,
      };
    }
    isCatching.value = false;
  }, 2000);
};

const closeMessage = () => {
  caughtMessage.value = null;
};

const goToPreviousPokemon = () => {
  if (pokemon.value && pokemon.value.id > 1) {
    fetchPokemonDetail(pokemon.value.id - 1);
  }
};

const goToNextPokemon = () => {
  if (pokemon.value) {
    fetchPokemonDetail(pokemon.value.id + 1);
  }
};

onMounted(() => {
  fetchPokemonDetail(route.params.id);
});
</script>

<template>


  <!-- <Navbar /> -->
  <icon/>
  <icon_side />
  <div
    :class="[
      'min-h-auto',
      'flex',
      'items-center',
      'justify-center',
      'p-5',
      'bg-gray-100',
    ]"
  >
    <div v-if="isLoading" class="text-center text-lg font-semibold text-white">
      <img src="../assets/pokego.png" class=" animate-spin" alt="">
    </div>
    <div
      v-else-if="errorMessage"
      class="text-center text-red-500 font-semibold"
    >
      {{ errorMessage }}
    </div>

    <div v-else class="w-full max-w-5xl">
      <div :class="[
      'rounded-3xl',
      'p-6',
      'w-full',
      'flex',
      'flex-col',
      'md:flex-row',
      'py-10',
      backgroundColor,
      ]" >
        <!-- ///////////// -->
        <div class="desktop-buttons">
          <button
            @click="goToPreviousPokemon"
            class="btn btn-neutral size-10 rounded-full mr-8  m-auto ml-auto bg-yellow-400"
          >
          <i class="fa-regular fa-circle-left"></i>
          </button>
        </div>

        <div
          class="md:w-1/2 flex flex-col items-center bg-transparent mr-3 p-4 rounded-2xl "
        >
          <h1 class="text-xl font-bold text-gray-100">{{ pokemon.id }}</h1>
          <img :src="pokemon.image" :alt="pokemon.name" class=" hover:scale-105  w-64 h-64 bg-[url('../assets/pokeS.png')] bg-no-repeat bg-cofer bg-[position:top_0px_right_0px] transition ease-in-out" />
          <h1 class="text-3xl font-bold text-gray-100 mt-4">
            {{ pokemon.name }}
          </h1>

          <button
            @click="catchPokemon"
            class="mt-4 px-6 py-2  rounded-full bg-yellow-400 duration-200 hover:bg-gray-400 flex items-center justify-center"
            :disabled="isCatching"
          >
            <img
              src="../assets/pokebal.png"
              class="animate-bounce w-8 h-8"
              alt="Pokeball"
            />
            <span v-if="!isCatching" class=" text-1xl mr-1 ml-1 font-semibold"
              >Catch {{ pokemon.name }}</span
            >
            <span v-else class="animate-bounce">Throwing Pokéball...</span>
          </button>
        </div>

        <div class=" m-auto w-auto bg-gray-600 rounded-3xl shadow-lg p-6 max-w-lg">
          <div class="flex gap-2">
            <button
              @click="activeTab = 'about'"
              :class="activeTab === 'about' ? 'bg-blue-600' : 'bg-gray-200'"
              class="px-3 py-1 rounded-md text-gray-800 font-semibold"
            >
              About
            </button>
            <button
              @click="activeTab = 'stats'"
              :class="activeTab === 'stats' ? 'bg-blue-600' : 'bg-gray-200'"
              class="px-3 py-1 rounded-md text-gray-800"
            >
              Stats
            </button>
            <button
              @click="activeTab = 'moves'"
              :class="activeTab === 'moves' ? 'bg-blue-600' : 'bg-gray-200'"
              class="px-3 py-1 rounded-md text-gray-800"
            >
              Moves
            </button>
          </div>

          <div v-if="activeTab === 'about'" class="mt-8 space-y-4 h-80">
            <p class="text-gray-400 font-mono font-semibold">
              <strong>Types:</strong>
              <span
                v-for="type in pokemon.types"
                :key="type"
                class="px-2 py-1 bg-yellow-400 text-slate-800 rounded mx-1 text-sm capitalize"
              >
                {{ type }}
              </span>
            </p>
            <p class="text-gray-400 font-mono font-semibold">
              <strong>Height:</strong> {{ pokemon.height }}
            </p>
            <p class="text-gray-400 font-mono">
              <strong>Weight:</strong> {{ pokemon.weight }}
            </p>
            <p class="text-gray-400 font-mono font-semibold">
              <strong>Abilities:</strong>
              <span
                v-for="ability in pokemon.abilities"
                :key="ability"
                class="px-2 py-1 bg-yellow-400 mt-4 text-slate-800 rounded mx-1 text-sm capitalize"
              >
                {{ ability }}
              </span>
            </p>
            <p class="text-gray-400 font-mono">
              <strong>XP:</strong> {{ pokemon.experience }} Exp
            </p>
          </div>
          <div v-if="activeTab === 'moves'" class="mt-2 space-y-2 h-80">
          <div
            class="grid grid-rows-0 grid-cols-3 lg:grid-cols-3 gap-3 h-80 pr-3 overflow-y-scroll -mr-3 "
          >
            <span
              v-for="move in pokemon.moves"
              :key="move.move.name"
              class="border border-dashed py-2 hover:border-yellow-400 text-center text-sm font-semibold font-mono border-gray-500 text-white"
            >
              {{ move.move.name }}
            </span>
          </div>
        </div>
        <div v-if="activeTab === 'stats'" class="mt-4 space-y-2 h-80">
          <div v-for="stat in pokemon.stats" :key="stat.stat.name">
            <p class="text-white font-mono">
              {{ stat.stat.name.toUpperCase() }}: {{ stat.base_stat }}
            </p>
            <div class="w-full bg-white rounded-full h-3">
              <div
                class="bg-yellow-400 h-3 rounded-full transition-all duration-300 overflow-hidden "
                :style="{ width:Math.min (stat.base_stat,100) + '%' }"
              ></div>
            </div>
            </div>
            </div>
        </div>

        <div class="desktop-buttons">
          <button
            @click="goToNextPokemon"
            class="btn btn-neutral size-10 rounded-full m-auto ml-auto bg-yellow-400"
          >
          <i class="fa-regular fa-circle-right "></i>
          </button>
        </div>
      </div>

      <div class="mobile-buttons mt-4 flex justify-center space-x-4 ">
        <button
          @click="goToPreviousPokemon"
          class="btn btn-neutral size-10 rounded-full  m-auto ml-auto bg-yellow-400"
        >
        <i class="fa-regular fa-circle-left"></i>
        </button>
        <button
          @click="goToNextPokemon"
          class="btn btn-neutral size-10 rounded-full  m-auto ml-auto bg-yellow-400"
        >
        <i class="fa-regular fa-circle-right"></i>
        </button>
      </div>
    </div>
  </div>
<!-- ///////////// -->
  <div
    v-if="caughtMessage"
    class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 animate-fadeIn"
  >
    <div class="bg-white p-6 rounded-lg shadow-lg text-center w-96 relative">
      <button
        @click="closeMessage"
        class="absolute top-2 right-2 text-gray-500 hover:text-gray-700"
      >
        ✖
      </button>

      <div v-if="caughtMessage.success" class="animate-fadeIn">
        <h2 class="text-xl font-semibold text-green-700">
          Kamu Mendapatkan {{ caughtMessage.pokemon.name }}!
        </h2>
        <img
          :src="caughtMessage.pokemon.image"
          alt="Pokemon"
          class="w-32 mx-auto my-2"
        />
        <p class="text-gray-600">
          Type: {{ caughtMessage.pokemon.types.join(", ") }}
        </p>
      </div>
      <div v-else class="animate-fadeIn">
        <h2 class="text-xl font-semibold text-red-700">
          Pokemon Mu Pergi...
        </h2>
      </div>

      <button
        @click="closeMessage"
        class="mt-4 px-4 py-2 bg-gray-500 rounded hover:bg-red-600 text-white"
      >
        Close
      </button>
    </div>
  </div>
</template>

<style scoped>
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
.animate-fadeIn {
  animation: fadeIn 0.3s ease-in-out;
}

.mobile-buttons {
  display: none;
}

.desktop-buttons {
  display: flex;
}

@media (max-width: 768px) {
  .mobile-buttons {
    display: flex;
    justify-content: center;
    gap: 1rem;
    margin-top: 1rem;
  }

  .desktop-buttons {
    display: none;
  }
}
</style>
