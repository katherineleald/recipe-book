<script>
  // Import necessary styles and icons
  import "../app.css";
  import { faPlayCircle, faXmark } from "@fortawesome/free-solid-svg-icons";
  import { FontAwesomeIcon } from "@fortawesome/svelte-fontawesome";

  // Export properties for controlling modal visibility and selected meal data
  export let showModal; // boolean to show or hide the modal
  export let selectedMeal; // object representing the selected meal

  let dialog; // Reference to the HTMLDialogElement

  // Show the dialog when showModal is true
  $: if (dialog && showModal) dialog.showModal();

  // Function to get ingredients from the selected meal
  function getIngredients(meal) {
    return Array.from({ length: 20 }, (_, i) => ({
      ingredient: meal[`strIngredient${i + 1}`],
      measure: meal[`strMeasure${i + 1}`],
    })).filter((item) => item.ingredient); // Filter out empty ingredients
  }
</script>

<!-- svelte-ignore a11y-no-noninteractive-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<!-- svelte-ignore a11y-click-events-have-key-events -->
<dialog
  bind:this={dialog}
  on:close={() => (showModal = false)}
  on:click|self={() => dialog.close()}
>
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div on:click|stopPropagation class="p-5 bg-[#f2e9e4]">
    <div class="w-full flex justify-end">
      <!-- svelte-ignore a11y-autofocus -->
      <button
        autofocus
        on:click={() => dialog.close()}
        class="text-gray-700 hover:text-black transition ease-in-out"
      >
        <FontAwesomeIcon icon={faXmark} class="w-6 h-6" />
      </button>
    </div>
    {#if selectedMeal}
      <div class="flex flex-col sm:flex-row gap-4">
        <slot name="header">
          <div
            id="first-column"
            class="flex-shrink-0 sm:w-1/2 flex flex-col items-center"
          >
            <h2
              id="recipe__info__title"
              class="font-['Poppins'] font-semibold text-[1.75em] text-[#3d5a40] text-center pb-3 mb-2 leading-8 block sm:hidden"
            >
              {selectedMeal.strMeal}
            </h2>
            <img
              src={selectedMeal.strMealThumb}
              alt={selectedMeal.strMeal}
              class="rounded-lg w-11/12"
            />
            <div
              class="grid gap-4 grid-col-1 md:grid-cols-2 items-center w-11/12"
            >
              <div>
                <h4 class="mt-3">
                  <span class="font-semibold">Category:</span>
                  {selectedMeal.strCategory}
                </h4>
                <h4 class="mt-2">
                  <span class="font-semibold">Origin:</span>
                  {selectedMeal.strArea}
                </h4>
              </div>
              <div class="w-full justify-end hidden sm:flex">
                <a
                  href={selectedMeal.strYoutube}
                  target="_blank"
                  class="mt-3 bg-red-500 px-5 py-4 text-white rounded-lg hover:bg-red-800 transition ease-in-out font-['Poppins'] flex items-center"
                >
                  <FontAwesomeIcon icon={faPlayCircle} class="w-5 h-5 me-2" /> Watch
                  Tutorial
                </a>
              </div>
            </div>
          </div>
        </slot>
        <div
          id="second-column"
          class="sm:w-1/2 sm:overflow-auto sm:h-[80vh] px-3 sm:px-6"
        >
          <div id="recipe__info">
            <h2
              id="recipe__info__title"
              class="hidden sm:block font-['Poppins'] font-semibold text-[1.75em] text-[#3d5a40] text-center m-0 pb-3 leading-8"
            >
              {selectedMeal.strMeal}
            </h2>
            <div
              id="recipe__info__ingredients"
              class="font-['Noto_Sans'] text-[1.05em] my-2"
            >
              <span class="font-semibold text-[20px]">Ingredients:</span>
              <ul>
                {#each getIngredients(selectedMeal) as { ingredient, measure }}
                  <li class="list-disc">{measure} of {ingredient}</li>
                {/each}
              </ul>
            </div>
            <p
              id="recipe__info__instructions"
              class="font-['Noto_Sans'] text-[1.05em] text-justify mt-3"
            >
              <span class="font-semibold text-[20px]">Instructions:<br /></span>
              {selectedMeal.strInstructions}
            </p>
          </div>
          <div class="w-full flex justify-end sm:hidden mt-3">
            <a
              href={selectedMeal.strYoutube}
              target="_blank"
              class="mt-3 bg-red-500 px-5 py-4 text-white rounded-lg hover:bg-red-800 transition ease-in-out font-['Poppins'] flex items-center"
            >
              <FontAwesomeIcon icon={faPlayCircle} class="w-5 h-5 me-2" /> Watch
              Tutorial
            </a>
          </div>
        </div>
      </div>
    {/if}
  </div>
</dialog>

<style>
  /* Styles for the dialog */
  dialog {
    width: 75em;
    border-radius: 0.2em;
    border: none;
    padding: 0;
  }
  dialog::backdrop {
    background: rgba(0, 0, 0, 0.3);
  }
  dialog[open] {
    animation: zoom 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  }
  @keyframes zoom {
    from {
      transform: scale(0.95);
    }
    to {
      transform: scale(1);
    }
  }
  dialog[open]::backdrop {
    animation: fade 0.2s ease-out;
  }
  @keyframes fade {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
</style>
