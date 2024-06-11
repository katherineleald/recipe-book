<script>
  export let size = "50px";
</script>

<div
  class="spinner border-4 border-gray-200 border-t-[#3d5a40] rounded-full"
  style="width: {size}; height: {size};"
></div>

<style>
  .spinner {
    animation: spin 1s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }
</style>
