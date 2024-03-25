<script setup>

import { onMounted, ref, computed } from 'vue'
import axios from 'axios'

const ingredients = ref([]);
const selectedIngredient = ref(null);
const searchQuery = ref('');
let countdownInterval = null;
let remainingTime = ref(null);

function showIngredientDetails(ingredient) {
  remainingTime.value = ingredient.time;
  selectedIngredient.value = ingredient;
  clearInterval(countdownInterval);
}

async function getIngredients() {
  const response = await axios.get('http://localhost:8000/api/ingredients');
  ingredients.value = response.data['hydra:member'];
}

const filteredIngredients = computed(() => {
  const normalizedQuery = searchQuery.value.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
  return ingredients.value.filter(ingredient => {
    const normalizedIngredientName = ingredient.name.toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
    return normalizedIngredientName.includes(normalizedQuery);
  });
});

const startCountdown = () => {
      if (selectedIngredient.value.time) {
        clearInterval(countdownInterval);
        remainingTime.value = selectedIngredient.value.time;
        countdownInterval = setInterval(() => {
          remainingTime.value--;
          if (remainingTime.value === 0) {
            clearInterval(countdownInterval);
          }
        }, 1000); // Update countdown every second (1000 milliseconds)
      }
    };



onMounted(() => {
  getIngredients();
});

</script>

<template>
  <div class="bg-gradient-to-r from-violet-100 to-indigo-100 flex items-center justify-center">
    <div class="backdrop-blur-sm bg-white/40 p-6 rounded-lg shadow-sm border-violet-200 border">
      <div class="w-full flex justify-between items-center p-3">
        <h2 class="text-xl font-semibold text-indigo-900">C'est l'heure de cuisiner<br>oui mais combien de temps ?</h2>
        <button id="openModalBtn"
          class="flex items-center bg-gradient-to-r from-violet-300 to-indigo-300  border border-fuchsia-00 hover:border-violet-100 text-white font-semibold py-2 px-4 rounded-md transition-colors duration-300">
          <svg class="w-4 h-4 mr-2 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewbox="0 0 24 24"
            stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
          </svg>
          <p class="text-white">Nouvel ingrédient</p>
        </button>
      </div>

      <div class="w-full grid grid-cols-3 gap-8 justify-center p-1 mb-4" v-if="selectedIngredient">
        <div class="border rounded-lg overflow-hidden mt-4 p-2 border-indigo-500">
          <h2 class="text-2xl font-bold mb-4 text-indigo-900 uppercase">{{ selectedIngredient.name }}</h2>
          <p class="text-indigo-400 mb-4">{{ selectedIngredient.instructions }}</p>
        </div>
        <div class="border rounded-lg overflow-hidden mt-4 p-2 border-indigo-500">
          <p class="text-lg text-indigo-900">Durée: <span class="font-semibold">{{ selectedIngredient.time }}s</span>
          </p>
          <p class="text-lg text-indigo-900">Mesure: <span class="font-semibold">{{ selectedIngredient.weight }}g
            </span></p>
        </div>
        <div class="border rounded-lg overflow-hidden mt-4 p-2 border-indigo-500 flex flex-col">
          <p class="text-lg text-indigo-900 text-center mb-4">Démarrer la cuisson</p>
          <button @click="startCountdown"
            class="bg-gradient-to-r from-violet-300 to-indigo-300 border border-indigo-100 hover:border-violet-900 text-white font-semibold py-2 px-2 rounded-md transition-colors duration-300">{{remainingTime + " s"}}</button></div>
      </div>

      <!-- <div class="text-indigo-500" id="showIngredient">{{ component('show_ingredient')}}</div> -->
      <div class="relative w-full">
        <input type="search" v-model="searchQuery"
          class="w-full backdrop-blur-sm bg-white/20 py-2 pl-10 pr-4 rounded-lg focus:outline-none border-2 border-gray-200 focus:border-violet-300 transition-colors duration-300 text-indigo-800"
          placeholder="Trouver un ingredient...">
        <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
          <svg class="w-4 h-4 text-indigo-800 " aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
            viewbox="0 0 20 20">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
              d="m19 19-4-4m0-7A7 7 0 1 1 1 8a7 7 0 0 1 14 0Z" />
          </svg>
        </div>
      </div>


      <div class="w-full flex justify-center p-1 mb-4">
        <div class="border rounded-lg overflow-hidden mt-4">
          <table class="min-w-full">
            <thead class="text-xs text-gray-700 uppercase bg-gradient-to-r from-violet-300 to-indigo-300">
              <tr>
                <th scope="col" class="py-3 px-6">Ingrédients</th>
                <th scope="col" class="py-3 px-6">Durée</th>
                <th scope="col" class="py-3 px-6">Poids</th>
                <th scope="col" class="py-3 px-6">Catégorie</th>
                <th scope="col" class="py-3 px-6">Instructions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="ingredient in filteredIngredients" :key="ingredient.id"
                class="bg-white border-b text-indigo-700 hover:bg-indigo-200"
                @click="showIngredientDetails(ingredient)">
                <td class="py-4 px-6">{{ ingredient.name }}</td>
                <td class="py-4 px-6">{{ ingredient.time }}</td>
                <td class="py-4 px-6">{{ ingredient.weight }}</td>
                <td class="py-4 px-6">-</td>
                <td class="py-4 px-6">{{ ingredient.instructions }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>

</template>

<style scoped>

</style>
