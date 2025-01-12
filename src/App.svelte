<script>
import '@mux/mux-video';
import { writable } from 'svelte/store';

let userInteractions = writable([]);
let isPaused = true;
let videoElement;

function toggleVideoPlayback() {
    if (videoElement) {
      if (videoElement.paused) {
        videoElement.play();
        isPaused = false;
      } else {
        videoElement.pause();
        isPaused = true;
      }
    }
  }

function handleVideoClick(event) {
  const time = event.target.currentTime;
  const x = event.offsetX;
  const y = event.offsetY;
  const eventType = event.type === 'keydown' ? event.code : event.type;

  userInteractions.update(arr => [...arr, {time: formatTime(time), x, y, eventType: formatEvent(eventType)}])
  console.log($userInteractions);

  //**
  // *Using the Commented out section below we can confirm that offsetX and offsetY are
  // positions relative to video not page.
  // */
  // if (videoElement) {
  //     // Get the bounding rectangle of the video element
  //     const rect = videoElement.getBoundingClientRect();
      
  //     // Calculate the x and y position relative to the video
  //     const x = event.clientX - rect.left;
  //     const y = event.clientY - rect.top;
      
  //     console.log(`Mouse position relative to video: X=${x}, Y=${y}`);
  //   }
}

function formatEvent(eventType) {
  return eventType
    .replace(/([a-z])([A-Z])/g, '$1 $2')
    .replace(/\s+/g, ' ')
    .toLowerCase()
    .replace(/\b\w/g, (match) => match.toUpperCase());
}

function formatTime(time) {
  const minutes = Math.floor(time / 60);
  const seconds = Math.floor(time % 60);

  const minutesStr = minutes < 10 ? `0${minutes}` : `${minutes}`; 
  const secondsStr = seconds < 10 ? `0${seconds}` : `${seconds}`; 

  return `${minutesStr}:${secondsStr}`
}
</script>

<main class="h-screen">
  <div class="flex items-center h-full m-0">
    <!-- Video Section -->
    <div class="w-3/4 h-full flex items-center justify-center flex-col">
      <mux-video
        playback-id="DS00Spx1CV902MCtPj5WknGlR102V5HFkDe"
        metadata-video-title="Mad Max Fury Road"
        metadata-viewer-user-id="user-id-1234"
        controls
        class="m-4"
        onclick={handleVideoClick}
        onkeydown={handleVideoClick}
        bind:this={videoElement}
        tabindex="0"
        role="button"
      ></mux-video>
      <div class="m-4 w-[90%] h-32 bg-gray-800 text-white border border-gray-700 rounded-lg shadow-lg flex justify-center items-center">
        <button
          class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded"
          onclick={toggleVideoPlayback}
        >
          {isPaused ? "Play" : "Pause"}
        </button>
      </div>
    </div>

    <!-- Data Section -->
    <div class="m-4 w-1/4 h-3/4 bg-gray-800 text-white border border-gray-700 rounded-lg shadow-lg overflow-hidden overflow-y-auto scrollable-content">
      {#each $userInteractions as interaction}
        <div class="m-4">
          <div>
            {interaction.time}
          </div>
          <p>
            {#if interaction.eventType.toLowerCase() === 'click'}
              {`User clicked video at positions: X:${interaction.x}, Y:${interaction.y}`}
            {:else}
              {`User pressed ${interaction.eventType}`}
            {/if}
          </p>
        </div>
      {/each}
    </div>
  </div>
</main>


<style>
  .scrollable-content::-webkit-scrollbar {
    width: 8px; 
  }

  .scrollable-content::-webkit-scrollbar-track {
    background: #2d3748;
    border-radius: 4px; 
  }

  .scrollable-content::-webkit-scrollbar-thumb {
    background: #718096; 
    border-radius: 4px; 
  }

  .scrollable-content::-webkit-scrollbar-thumb:hover {
    background: #a0aec0; 
  }
</style>