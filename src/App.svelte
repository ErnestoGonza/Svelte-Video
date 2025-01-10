<script>
import { Video, Card } from 'flowbite-svelte';
</script>

<main class="h-screen">
  <div class="flex items-start h-full m-4">
    <!-- Video Section -->
    <div class="w-3/4 h-full flex items-center justify-center flex-col">
      <Video src="/assets/boat.mp4" controls class="w-full h-3/4 rounded-lg"></Video>
      <div class="m-4 w-[90%] h-40 bg-gray-800 text-white border border-gray-700 rounded-lg shadow-lg flex justify-center items-center">
        Play <!--TODO Create Play Button Card-->
      </div>
    </div>

    <!-- Data Section -->
    <div class="m-4 w-1/4 h-5/6 bg-gray-800 text-white border border-gray-700 rounded-lg shadow-lg">
      Data
    </div>
  </div>
</main>


<style>
 
</style>
