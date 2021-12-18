<script setup lang="ts">
import { ref, computed } from 'vue'

const searchValue = ref<string>( '' )
const pokemonData = ref<PokemonDataInterface | null>( null )
const idWithLeadingZeros = computed( () => {
  if ( pokemonData.value ) {
    return `00${ pokemonData.value.id }`.slice( -3 )
  }
  return null
})

interface TypeInterface {
  type: {
    name: string
  }
}

interface PokemonDataInterface {
  id: string
  name: string
  types: TypeInterface[]
  sprites: {
    front_default: string
  }
}

const fetchPokemonData = async () => {
  const apiUrl = 'https://pokeapi.co/api/v2'
  try {
    const response = await fetch( `${ apiUrl }/pokemon/${ searchValue.value }` )
    const data = await response.json()
    pokemonData.value = data
  } catch ( error ) {
    console.error( error )
  }
}
</script>

<template>
  <h1>Vue Pokédex App</h1>
  <form @submit.prevent="fetchPokemonData">
    <input
      type="text"
      v-model="searchValue"
      placeholder="Type in the name of your Pokémon…"
    />
    <button type="submit">Search</button>
  </form>

  <div v-if="pokemonData">
    <img
      :src="pokemonData.sprites.front_default"
      width="96"
      height="96"
      :alt="pokemonData.name"
    />
    <p>#{{ idWithLeadingZeros }}</p>
    <h2>{{ pokemonData.name }}</h2>
    <ul v-for="{ type } in pokemonData.types" :key="type.name">
      <li>{{ type.name }}</li>
    </ul>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
