<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'

const searchValue = ref<string>( '' )
const pokemonData = ref<PokemonDataInterface | null>( null )
const idWithLeadingZeros = computed( () => {
  if ( pokemonData.value ) {
    return `00${ pokemonData.value.id }`.slice( -3 )
  }
  return null
})
const capitalizedName = computed( () => {
  if ( pokemonData.value?.name ) {
    return (
      pokemonData.value.name.charAt( 0 ).toUpperCase() +
      pokemonData.value.name.slice( 1 )
    )
  }
  return ''
})

interface TypeInterface {
  type: {
    name: string
    url: string
  }
}

interface PokemonDataInterface {
  id: number
  name: string
  types: TypeInterface[]
  sprites: {
    front_default: string
  }
}

const fetchPokemonData = async (
  event: Event,
  queryType = 'pokemon',
  queryTerm = searchValue.value,
) => {
  const apiUrl = 'https://pokeapi.co/api/v2'
  try {
    const response = await fetch(
      `${ apiUrl }/${ queryType }/${ queryTerm.toLowerCase() }`,
    )
    const data = await response.json()
    pokemonData.value = data
  } catch ( error ) {
    console.error( error )
    pokemonData.value = null
  }
}

const selectAdjacentPokemon = ( event: Event, position: string ) => {
  const queryTerm = () => {
    if ( pokemonData.value?.id ) {
      if ( position === 'prev' && pokemonData.value.id === 1 ) return '898'
      if ( position === 'next' && pokemonData.value.id === 898 ) return '1'
      if ( position === 'prev' ) return ( pokemonData.value.id - 1 ).toString()
      if ( position === 'next' ) return ( pokemonData.value.id + 1 ).toString()
    }
    return ''
  }
  fetchPokemonData( event, 'pokemon', queryTerm() )
  searchValue.value = queryTerm()
}

onMounted( () => {
  window.addEventListener( 'keydown', ( event: KeyboardEvent ) => {
    const target = event.target as HTMLElement
    if ( !pokemonData.value || target.tagName === 'INPUT' ) return

    switch ( event.key ) {
      case 'ArrowLeft':
        selectAdjacentPokemon( event, 'prev' )
        break
      case 'ArrowRight':
        selectAdjacentPokemon( event, 'next' )
        break
      default:
        break
    }
  })
})
</script>

<template>
  <header class="site-header">
    <div class="container">
      <h1 class="site-heading">Vue Pokédex App</h1>
    </div>
  </header>
  <main class="site-main">
    <div class="container">
      <form @submit.prevent="fetchPokemonData" class="search-form">
        <div class="search-box">
          <input
            type="text"
            class="search-box__input"
            v-model="searchValue"
            placeholder="Type in the name of your Pokémon…"
          />
          <button type="submit" class="search-box__button">Search</button>
        </div>
      </form>
      <div v-if="pokemonData" class="search-result">
        <img
          :src="pokemonData.sprites.front_default"
          class="search-result__image"
          width="96"
          height="96"
          :alt="capitalizedName"
        />
        <p class="search-result__id">#{{ idWithLeadingZeros }}</p>
        <h2 class="search-result__name">{{ capitalizedName }}</h2>
        <ul class="search-result__types">
          <li v-for="{ type } in pokemonData.types" :key="type.name">
            {{ type.name }}
          </li>
        </ul>
        <button @click="selectAdjacentPokemon($event, 'prev')">Previous</button>
        <button @click="selectAdjacentPokemon($event, 'next')">Next</button>
      </div>
    </div>
  </main>
  <footer class="site-footer">
    <div class="container">
      <a
        href="https://github.com/rodrigodagostino/vue-pokedex-app"
        target="_blank"
      >
        Made with <img src="./assets/vue-logo.png" alt="Vue.js logo" /> by
        Rodrigo D’Agostino
      </a>
    </div>
  </footer>
</template>

<style lang="scss">
/**
 * Variables
 */
:root {
  --font-main: 'Baloo 2', Avenir, Helvetica, Arial, sans-serif;

  --color-main--lightest: var(--rose-200);
  --color-main--lighter: var(--rose-300);
  --color-main--light: var(--rose-400);
  --color-main: var(--rose-500);
  --color-main--dark: var(--rose-600);
  --color-main--darker: var(--rose-700);
  --color-main--darkest: var(--rose-800);

  --white: #f3f3f5;
  --gray-050: #ededf0;
  --gray-100: #e1e1e6;
  --gray-150: #d2d2da;
  --gray-200: #c3c3ce;
  --gray-300: #a4a6b5;
  --gray-400: #86889d;
  --gray-500: #686a84;
  --gray-600: #585a70;
  --gray-650: #515266;
  --gray-700: #494a5c;
  --gray-750: #414253;
  --gray-800: #393a49;
  --gray-850: #31323f;
  --gray-900: #2a2a35;
  --gray-950: #22222b;
  --black: #15151a;

  --red-050: #fef2f2;
  --red-100: #fee2e2;
  --red-200: #fecaca;
  --red-300: #fca5a5;
  --red-400: #f87171;
  --red-500: #ef4444;
  --red-600: #dc2626;
  --red-700: #b91c1c;
  --red-800: #991b1b;
  --red-900: #7f1d1d;

  --rose-050: #fff1f2;
  --rose-100: #ffe4e6;
  --rose-200: #fecdd3;
  --rose-300: #fda4af;
  --rose-400: #fb7185;
  --rose-500: #f43f5e;
  --rose-600: #e11d48;
  --rose-700: #be123c;
  --rose-800: #9f1239;
  --rose-900: #881337;
}

/**
 * Reset styles.
 */
html {
  box-sizing: border-box;
  height: 100%;
}

*,
*::before,
*::after {
  // Inherit box-sizing to make it easier to change the property
  // for components that leverage other behavior
  // http://css-tricks.com/inheriting-box-sizing-probably-slightly-better-best-practice/
  box-sizing: inherit;
}

/* Reset margins and paddings on most elements */
body,
h1,
h2,
h3,
h4,
h5,
h6,
ul,
ol,
li,
p,
pre,
blockquote,
figure,
hr {
  margin: 0;
  padding: 0;
}

a {
  color: var(--white);
  transition: outline 0.32s ease;
  outline: 1px dotted transparent;
}

a:focus {
  outline: 1px dotted var(--white);
}

/* Reset forms and buttons */
input,
textarea,
select,
button {
  color: inherit;
  font: inherit;
  letter-spacing: inherit;
}

input,
textarea,
button {
  border: 1px solid gray;
  min-width: 0;
}

button {
  border-radius: 0;
  padding: 0.75em 1em;
  background-color: transparent;
}

button * {
  pointer-events: none;
}

/* Easy responsive for media elements */
embed,
iframe,
img,
object,
video {
  display: block;
  max-width: 100%;
}

/* Useful table styles */
table {
  table-layout: fixed;
  width: 100%;
}

/**
 * Base styles.
 */
body {
  font-family: var(--font-main);
  font-size: 1rem;
  color: var(--gray-600);
  background-color: var(--gray-100);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  height: 100%;
  display: flex;
  justify-content: center;
}

.site {
  display: flex;
  flex-direction: column;
  max-width: 100%;
  min-height: 100vh;
}

.container {
  display: flex;
  max-width: 52rem;
  margin: 0 auto;
  padding: 1.25rem;
  position: relative;
}

.site-header {
  .container {
    display: block;
  }

  .site-heading {
    font-size: 3rem;
    font-weight: 700;
    text-align: center;
  }
}

.site-main {
  .container {
    flex-direction: column;
  }
}

.search-form {
  width: 100%;
  transition: margin 0.32s ease;
}

.search-box {
  display: flex;
  max-width: 100%;
  margin: 0 auto;
  background-color: var(--gray-150);
  border-radius: 0.75rem;
  font-size: 1.125rem;
  outline: 2px solid transparent;
  transition: background-color 0.32s ease, outline 0.32s ease,
    max-width 0.32s ease;
}

.search-box:focus-within {
  background-color: var(--gray-050);
  outline: 2px solid var(--color-main);
}

.search-box__input {
  background-color: transparent;
  border: none;
  padding: 1rem;
  flex: 1;
  outline: none;
  min-width: 0; /* Reset default property. */
}

.search-box__button {
  color: var(--color-main);
  border: none;
  border-radius: 0 0.75rem 0.75rem 0;
  outline: none;
  transition: color 0.32s ease, background-color 0.32s ease;
  padding: 0.75rem 1rem;
  cursor: pointer;
}

.search-box__button:focus,
.search-box__button:hover {
  color: var(--white);
  background-color: var(--color-main);
}

.search-result {
  padding: 2rem 0;
  text-align: center;

  .search-result__image {
    margin: 0 auto;
  }

  .search-result__types {
    display: flex;
    justify-content: center;
    list-style-type: none;

    li + li {
      margin-left: 0.75rem;
    }
  }
}

@media screen and (min-width: 30em) {
  .search-box__input {
    padding: 1rem 1.5rem;
  }

  .search-box__button {
    padding: 1rem 1.5rem;
  }
}

@media screen and (min-width: 45em) {
  .search-box {
    max-width: 80%;
  }
}

.site-footer {
  font-size: 0.875rem;
  text-align: center;
  margin-top: auto;
}

.site-footer .container {
  display: flex;
  justify-content: center;
  padding: 1rem 0;
}

.site-footer a {
  color: var(--gray-400);
  text-decoration: none;
  transition: color 0.24s ease;
}

.site-footer a:focus,
.site-footer a:hover {
  color: var(--gray-600);
}

.site-footer img {
  height: 1.25rem;
  width: auto;
  vertical-align: middle;
  margin: 0 0.25rem;
  display: inline-block;
}
</style>
