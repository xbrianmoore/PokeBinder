<script setup>
import { ref, onMounted, computed } from 'vue'

const pokemon = ref({
  name: 'Pikachu',
  type: 'Electric',
  hp: 60
})

const card = ref(null)
const cardImageUrl = computed(() => {
  return card.value ? card.value.image + '/high.webp' : ''
})
onMounted(async () =>{
  const response = await fetch('https://api.tcgdex.net/v2/en/cards/swsh3-136')
  const data = await response.json()
  card.value = data
})
/*  --- JSON OBJECT EXAMPLE ---
{
  "category": "Pokemon",
  "id": "swsh3-136",
  "illustrator": "tetsuya koizumi",
  "image": "https://assets.tcgdex.net/en/swsh/swsh3/136",
  "localId": "136",
  "name": "Furret",
  "rarity": "Uncommon",
  "set": {
    "cardCount": {
      "official": 189,
      "total": 201
    },
    "id": "swsh3",
    "logo": "https://assets.tcgdex.net/en/swsh/swsh3/logo",
    "name": "Darkness Ablaze",
    "symbol": "https://assets.tcgdex.net/univ/swsh/swsh3/symbol"
  },
  "variants": {
    "firstEdition": false,
    "holo": false,
    "normal": true,
    "reverse": true,
    "wPromo": false
  },
  "hp": 110,
  "types": [
    "Colorless"
  ],
  "evolveFrom": "Sentret",
  "description": "It makes a nest to suit its long and skinny body. The nest is impossible for other Pokémon to enter.",
  "stage": "Stage1",
  "attacks": [
    {
    "cost": [
      "Colorless"
    ],
    "name": "Feelin' Fine",
    "effect": "Draw 3 cards."
    },
    {
    "cost": [
      "Colorless"
    ],
    "name": "Tail Smash",
    "effect": "Flip a coin. If tails, this attack does nothing.",
    "damage": 90
    }
  ],
  "weaknesses": [
    {
      "type": "Fighting",
      "value": "×2"
    }
  ],
  "retreat": 1,
  "regulationMark": "D",
  "legal": {
    "standard": true,
    "expanded": true
  }
}
*/


</script>

<template>
// implement search bar here
// debounce search results through a time buffer
//on enter, show list of results with card preview, name, set, and rarity
  <div v-if="card">
    <h1> {{ card.name }}</h1>
    <img :src='cardImageUrl' alt="card.name">
    <p> {{ card.hp }}</p>
    <p v-for="weaknesses in card.weaknesses" :key="weaknesses.type"> 
      Weak to {{ weaknesses.type }} by {{ weaknesses.value }}</p> 
    </div>
  <div v-else>
    <h1>Loading...</h1>

 </div>
</template>

<style>
body{
    background-color: rgb(49, 49, 51);
    margin: 0;
    padding: 20px;
}

h1{
  color: rgb(255, 255, 255);
}
p{
  color: beige;
}
</style>
