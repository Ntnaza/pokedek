<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";

const pokemonList = ref([]);
const showCount = ref(20);
const router = useRouter();

const fetchPokemon = async () => {
  try {
    const promises = [];
    for (let i = 1; i <= 200; i++) {
      promises.push(
        fetch(`https://pokeapi.co/api/v2/pokemon/${i}`).then((res) =>
          res.json()
        )
      );
    }

    const results = await Promise.all(promises);
    pokemonList.value = results.map((pokemon) => ({
      id: pokemon.id,
      name: pokemon.name,
      image: pokemon.sprites.other["official-artwork"].front_default,
    }));
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

onMounted(fetchPokemon);

const showMore = () => {
  showCount.value += 20;
};

const goToDetail = (id) => {
  router.push(`/pokemon/${id}`);
};
</script>

<template>
    <section
    class="grid  md:grid-cols- lg:grid-cols-4 gap-3 p-3 -mt-25"
    >
    <div v-if="isLoading" class="text-center text-lg font-semibold text-white">
      Loading...
    </div>
    <div
    v-for="pokemon in pokemonList.slice(0, showCount)"
    :key="pokemon.id"
    class="bg-transparant transition bg-[url('../assets/pokeS.png')] bg-no-repeat bg-cofer bg-[position:top_10px_right_20px] ease-in-out hover:-translate-y-2 rounded-xl p-4 flex flex-col items-center group"
    @click="goToDetail(pokemon.id)"
    >
    <div class="text-yellow-400 text-sm font-semibold hover:text-blue-400 transition-colors">#{{ pokemon.id }}</div>
    <img
    :src="pokemon.image"
    alt="Pokemon"
    class="w-64 h-64 object-contain"
    />
    <h1 class="text-xl font-bold text-gray-300 mt-2 capitalize transition-colors group-hover:text-yellow-400">
        {{ pokemon.name }}
    </h1>
</div>

</section>
<footer class="flex flex-col items-center justify-center py-4">
    
    
    <div class="px-7 py-2 text-l mb-2 bg-gray-400 font-semibold text-white hover:bg-yellow-400 transition flex items-center justfy-center rounded-lg text-xl" @click="showMore">
      <img src="../assets/pika.png" class="h-7 mr-3 animate-pulse" alt="">
        <button
        @click="showMore"
        class=""
        >
        Show more..
    </button>
</div>

</footer>

</template>