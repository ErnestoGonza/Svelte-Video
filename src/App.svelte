<script>
import '@mux/mux-video';
import { writable } from 'svelte/store';
import Chart from 'chart.js/auto';
import { onMount } from 'svelte';
import { ThumbsDownSolid, ThumbsUpSolid } from 'flowbite-svelte-icons';

let userInteractions = writable([]);
let thumbsUp = writable(5);
let thumbsDown = writable(1);

let isPaused = true;
let videoElement;
let scrollableContent;
let chart;
let chartCanvas;

function handleVote(isThumbsUp) {
  isThumbsUp ? thumbsUp.update(num => num += 1) : thumbsDown.update(num => num += 1);
  const time = videoElement.currentTime;

  userInteractions.update(arr => [...arr, 
    {
      time: formatTime(time), 
      x: null, 
      y: null, 
      eventType: isThumbsUp ? formatEvent("Thumbs Up") : formatEvent("Thumbs Down"),
      currentRating: calculateRating(),
    }
  ])
  setScrollBarLocation();
}

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
  setScrollBarLocation();
}

function calculateRating() {
  return Math.round($thumbsUp / ($thumbsDown + $thumbsUp) * 100);
}

function setScrollBarLocation() {
  setTimeout(() => {
    return scrollableContent.scrollTop = scrollableContent.scrollHeight;
  }, 1)
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

function initializeChart() {
    if (chart) {
        chart.destroy();
    }
    chart = new Chart(chartCanvas, {
        type: 'doughnut',
        data: {
            datasets: [
                {
                    data: [$thumbsUp, $thumbsDown],
                    backgroundColor: ['#4caf50', '#F44336'], 
                    hoverBackgroundColor: ['#66bb6a', '#E57373'],
                },
            ],
        },
        options: {
            responsive: true,
            plugins: {
                legend: {
                    display: true,
                    position: 'top',
                },
            },
        },
    });
}

$: if (chart) {
  chart.data.datasets[0].data = [$thumbsUp, $thumbsDown];
  chart.update();
}

onMount(() => {
    initializeChart();
});
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
      <div class="m-4 w-[90%] h-32 bg-gray-800 text-white border border-gray-700 rounded-lg shadow-lg flex justify-evenly items-center">
        <div class="h-3/4">
          <canvas bind:this={chartCanvas}></canvas>
        </div>
        <button
          class="bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded"
          onclick={toggleVideoPlayback}
        >
          {isPaused ? "Play" : "Pause"}
        </button>
        <div class="h-2/4 w-28 flex justify-evenly">
          <button class="thumbsDown" onclick={() => handleVote(false)}>
            <ThumbsDownSolid class="h-9"/>
          </button>
          <button class="thumbsUp" onclick={() => handleVote(true)}>
            <ThumbsUpSolid class="h-9"/>
          </button>
        </div>
      </div>
    </div>

    <!-- Data Section -->
    <div 
      class="m-4 w-1/4 h-3/4 bg-gray-800 text-white border border-gray-700 rounded-lg shadow-lg overflow-hidden overflow-y-auto scrollable-content" bind:this={scrollableContent}
    >
      {#each $userInteractions as interaction}
        <div class="m-4">
          <div>
            {interaction.time}
          </div>
          <p>
            {#if interaction.eventType.toLowerCase() === 'click'}
                {`User clicked video at positions: X:${interaction.x}, Y:${interaction.y}`}
              {:else if interaction.eventType.toLowerCase().includes('thumbs')}
                {`User gave a ${interaction.eventType}, current approval rating: ${interaction.currentRating}%`}
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

  .thumbsUp {
    color: #4CAF50; 
    transition: color 0.3s ease; 
  }

  .thumbsUp:hover {
    color: #66bb6a;
  }

  .thumbsDown {
    color: #F44336; 
    transition: color 0.3s ease; 
  }

  .thumbsDown:hover {
    color: #E57373;
  }
</style>