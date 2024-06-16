<script>
  // Import necessary components and stores
  import Modal from "../components/Modal.svelte";
  import LoadingSpinner from "../components/LoadingSpinner.svelte";
  import { mealsStore } from "../stores/mealsStore";
  import { onMount } from "svelte";
  import { fade } from 'svelte/transition';
  import {
    faEye,
    faTrashAlt,
    faFaceSadTear,
  } from "@fortawesome/free-solid-svg-icons";
  import { FontAwesomeIcon } from "@fortawesome/svelte-fontawesome";

  // Define local state variables
  let showModal = false;
  let selectedMeal = null;
  let meals = [];
  let alphabet = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".split("");
  let activeLetter = "A";
  let isLoading = false;
  let removing = new Set(); // Set to track removing items

  // Subscribe to mealsStore to update meals variable
  mealsStore.subscribe((value) => {
    meals = value;
  });

  // Function to load meals based on the selected letter
  async function loadMeals(letter = "A") {
    isLoading = true;
    activeLetter = letter; // Update active letter
    const response = await fetch(
      `https://www.themealdb.com/api/json/v1/1/search.php?f=${letter.toLowerCase()}`
    );
    const data = await response.json();
    mealsStore.set(data.meals); // Update meals store with fetched data
    isLoading = false;
  }

  // Load meals when the component mounts
  onMount(() => {
    loadMeals(activeLetter); // Load meals for the active letter
  });

  // Function to get ingredients and their measure from a meal object
  function getIngredients(meal) {
    return Array.from({ length: 20 }, (_, i) => ({
      ingredient: meal[`strIngredient${i + 1}`],
      measure: meal[`strMeasure${i + 1}`],
    })).filter((item) => item.ingredient); // Filter out empty ingredients
  }

  // Function to open the modal with the selected meal
  function openModal(meal) {
    selectedMeal = meal;
    showModal = true;
  }

  // Function to close the modal
  function closeModal() {
    showModal = false;
    selectedMeal = null;
  }

  // Function to remove a meal from the meals store with fade-out animation
  function removeMeal(id) {
    removing.add(id);
    setTimeout(() => {
      mealsStore.update((currentMeals) => {
        return currentMeals.filter((meal) => meal.idMeal !== id);
      });
      removing.delete(id);
    });
  }
</script>

<!-- Header section with title and alphabet filter buttons -->
<div id="header" class="mt-3 sm:pb-14 pb-2">
  <h1
    id="title"
    class="text-center font-medium text-[#3d5a40] sm:text-7xl text-6xl pb-4 font-['Playball']"
  >
    Recipe Book
  </h1>
  <div class="flex flex-col mx-auto sm:px-3 px-5 w-fit">
    <h4 class="font-semibold text-lg font-['Noto_Sans']">
      Filter by first letter:
    </h4>
    <!-- Alphabet filter buttons -->
    <div class="alphabet-buttons flex flex-wrap justify-center">
      {#each alphabet as letter}
        <button
          on:click={() => loadMeals(letter)}
          class="m-0.5 p-2 px-3 bg-[#f2e9e4] border border-[#3d5a40] text-[#3d5a40] font-['Poppins'] text-base rounded cursor-pointer transition-colors duration-300 ease-in-out hover:bg-[#3d5a40] hover:text-[#f2e9e4] {letter ===
          activeLetter
            ? 'active'
            : ''}"
        >
          {letter}
        </button>
      {/each}
    </div>
  </div>
</div>

<!-- Meals display section -->
<div
  class="grid sm:grid-cols-2 lg:grid-cols-3 gap-x-5 gap-y-20 sm:m-5 sm:mt-5 mt-14 mb-5 px-5"
>
  {#if isLoading}
    <div class="col-span-full flex justify-center items-center">
      <LoadingSpinner size="100px" />
    </div>
  {:else}
    <!-- Meal card -->
    {#if meals}
      {#each meals as meal (meal.idMeal)}
        <div
          id="recipe__card"
          class="flex flex-col items-center justify-between bg-[#eadbd3] shadow-xl p-5 rounded-lg relative {removing.has(meal.idMeal) ? 'animate-fadeOut' : 'animate-fadeIn'}"
          transition:fade
        >
          <!-- Thumbnail that opens modal on click -->
          <!-- svelte-ignore a11y-click-events-have-key-events -->
          <!-- svelte-ignore a11y-no-static-element-interactions -->
          <a
            id="recipe__thumbnail--openModal"
            on:click={() => openModal(meal)}
            class="relative inline-block"
          >
            <img
              id="recipe__thumbnail"
              class="rounded-full shadow-xl cursor-pointer relative -top-[60px]"
              src={meal.strMealThumb}
              alt={meal.strMeal}
              width="300"
            />
            <div
              class="overlay absolute inset-0 flex items-center justify-center text-white opacity-0 mt-[-60px] h-full rounded-full cursor-pointer bg-[rgba(87,143,116,0.75)] transition-opacity duration-300 ease-in-out hover:opacity-1"
            >
              <FontAwesomeIcon icon={faEye} class="w-16 h-16" />
            </div>
          </a>
          <div id="recipe__info">
            <h2
              id="recipe__info__title"
              class="font-['Poppins'] font-semibold text-[1.75em] text-[#3d5a40] text-center mt-[-40px] pb-3 mb-2 leading-8"
            >
              {meal.strMeal}
            </h2>
            <!-- Ingredients: shows the first ingredientes with measures that can fit in one line -->
            <div
              class="recipe-ingredients line-clamp-1 w-full m-0 overflow-hidden"
            >
              <h3
                id="recipe__info__ingredients"
                class="font-['Noto_Sans'] font-[1.05em] mb-1 m-0"
              >
                <span class="font-semibold">Ingredients:</span>
                {meal.strMeasure1} of
                {meal.strIngredient1}, {meal.strMeasure2} of
                {meal.strIngredient2}, {meal.strMeasure3} of
                {meal.strIngredient3}, {meal.strMeasure4} of
                {meal.strIngredient4}, {meal.strMeasure5} of
                {meal.strIngredient5},
              </h3>
            </div>
            <!-- Instructions: only shows the first 3 lines of text -->
            <div
              class="recipe-instructions line-clamp-3 w-full m-0 overflow-hidden"
            >
              <p
                id="recipe__info__instructions"
                class="pt-2 font-['Noto_Sans'] font-[0.95em] text-justify m-0"
              >
                {meal.strInstructions}
              </p>
            </div>
          </div>
          <!-- Buttons Container -->
          <div
            id="recipe__button__container"
            class="mt-4 flex items-center justify-end w-full"
          >
            <!-- Delete Button: deletes meal -->
            <button
              on:click={() => removeMeal(meal.idMeal)}
              class="me-3 hover:text-red-500 transition ease-in-out"
            >
              <FontAwesomeIcon icon={faTrashAlt} class="w-8 h-8" />
            </button>
            <!-- Read More Button: opens modal -->
            <button
              on:click={() => openModal(meal)}
              class="bg-[#3d5a40] font-['Poppins'] p-3 text-white rounded-md hover:bg-[#1e2c20] transition ease-in-out"
              >Read More</button
            >
          </div>
        </div>
      {/each}
    {:else}
      <!-- Message that displays when there are no meals with the active letter -->
      <div class="flex flex-col col-span-full items-center font-['Noto_Sans'] text-center">
        <FontAwesomeIcon
          icon={faFaceSadTear}
          class="w-32 h-32 text-[#3d5a40] mb-5"
        />
        <h3 class="text-3xl mb-3 font-semibold">
          There are no meals that start with this letter
        </h3>
        <h5 class="text-xl">Try with another one!</h5>
      </div>
    {/if}
  {/if}
</div>

<!-- Modal component for displaying detailed meal information -->
<Modal bind:showModal {selectedMeal} on:close={closeModal} />

<div id="footer" class="w-full p-3 pt-5 bottom-0">
  <p class="text-center font-['Noto_Sans'] text-[0.85em]">&copy; 2024 Katherine Leal | All Rights Reserved</p>
</div>

<!-- Styles -->
<style lang="postcss">
  /* Importing Google Fonts */
  @import url("https://fonts.googleapis.com/css2?family=Noto+Sans:ital,wght@0,100..900;1,100..900&family=Playball&family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900&display=swap");

  /* Global styles */
  :global(html) {
    background: #f2e9e4;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  #recipe__thumbnail--openModal:hover .overlay {
    opacity: 1;
  }

  /* Active alphabet button styles */
  .alphabet-buttons button.active {
    background-color: #3d5a40;
    color: #f2e9e4;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  @keyframes fadeOut {
    from {
      opacity: 1;
    }
    to {
      opacity: 0;
    }
  }

  .animate-fadeIn {
    animation: fadeIn 0.5s ease-in-out;
  }

  .animate-fadeOut {
    animation: fadeOut 0.5s ease-in-out;
  }
</style>
