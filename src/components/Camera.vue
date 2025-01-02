<script setup>
import { ref, onMounted } from "vue";

const canvas = ref(null);
const video = ref(null);
const ctx = ref(null);
const filterMenuOpen = ref(false);

const constraints = ref({
  audio: false,
  video: { facingMode: "user" },
});

const filters = ref([
  { name: "Original", value: "none", icon: "🎨" },
  { name: "Vintage Warmth", value: "sepia(60%) brightness(110%)", icon: "🌅" },
  {
    name: "Cool Breeze",
    value: "hue-rotate(180deg) brightness(90%)",
    icon: "❄️",
  },
  { name: "Soft Glow", value: "brightness(120%) blur(2px)", icon: "✨" },
  { name: "Vivid Colors", value: "saturate(200%) contrast(120%)", icon: "🌈" },
  { name: "Dreamy Blur", value: "blur(5px) opacity(85%)", icon: "🌫️" },
  {
    name: "Golden Hour",
    value: "sepia(40%) hue-rotate(-20deg) brightness(120%)",
    icon: "🌞",
  },
  { name: "High Contrast", value: "contrast(250%) saturate(80%)", icon: "⚡" },
  { name: "Oceanic", value: "hue-rotate(200deg) saturate(150%)", icon: "🌊" },
  { name: "Pop Art", value: "invert(100%) hue-rotate(90deg)", icon: "🎨" },
]);

const selectedFilter = ref("none");
const countdown = ref(0);
const isCountdownActive = ref(false);
const flashVisible = ref(false);
const isFrontCamera = ref(true);

onMounted(async () => {
  if (video.value && canvas.value) {
    ctx.value = canvas.value.getContext("2d");

    // Set canvas size to match the video feed
    resizeCanvas();

    await navigator.mediaDevices
      .getUserMedia(constraints.value)
      .then(setStream)
      .catch((e) => {
        console.error(e);
      });
  }

  window.addEventListener("resize", resizeCanvas);
});

function resizeCanvas() {
  if (canvas.value && video.value) {
    const screenWidth = window.innerWidth;
    const screenHeight = screenWidth * (9 / 16); // Maintain a 16:9 aspect ratio
    canvas.value.width = screenWidth;
    canvas.value.height = screenHeight;
  }
}

function setStream(stream) {
  video.value.srcObject = stream;
  video.value.play();

  requestAnimationFrame(draw);
}

function draw() {
  ctx.value.save(); // Save the current canvas state
  ctx.value.scale(-1, 1); // Flip the canvas horizontally
  ctx.value.translate(-canvas.value.width, 0); // Adjust the drawing origin

  // Apply the selected filter
  ctx.value.filter = selectedFilter.value;

  ctx.value.drawImage(
    video.value,
    0,
    0,
    canvas.value.width,
    canvas.value.height
  );

  ctx.value.restore(); // Restore the canvas to its original state
  requestAnimationFrame(draw);
}

function triggerFlash() {
  flashVisible.value = true;
  setTimeout(() => {
    flashVisible.value = false;
  }, 300); // Flash effect duration
}

function takePicture() {
  triggerFlash(); // Trigger the flash effect
  const link = document.createElement("a");
  link.download = `vue-cam-${new Date().toISOString()}.png`;
  link.href = canvas.value.toDataURL();
  link.click();
}

// Countdown feature
function startCountdown() {
  isCountdownActive.value = true;
  countdown.value = 5;

  const countdownInterval = setInterval(() => {
    if (countdown.value > 0) {
      countdown.value--;
    } else {
      clearInterval(countdownInterval);
      takePicture(); // Take picture after countdown
      isCountdownActive.value = false; // Reset countdown state
    }
  }, 1000);
}

function getFilterImage(filterValue) {
  const filterImages = {
    none: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQCiZD12EU_Zm57G1wc72AaNVHGoLhQBIHPcg&s",
    "sepia(60%) brightness(110%)":
      "https://static.vecteezy.com/system/resources/previews/026/829/465/non_2x/beautiful-girl-with-autumn-leaves-photo.jpg",
    "hue-rotate(180deg) brightness(90%)":
      "https://static.vecteezy.com/system/resources/previews/031/563/389/non_2x/beautiful-asian-girl-in-flower-garden-ai-generative-photo.jpg",
    "brightness(120%) blur(2px)":
      "https://pics.craiyon.com/2023-09-09/94523cccbeb54c8992fdd5d246f0c6cf.webp",
    "saturate(200%) contrast(120%)":
      "https://i.pinimg.com/550x/38/09/c3/3809c319d6b40a4efda99bf5500fe6ef.jpg",
    "blur(5px) opacity(85%)":
      "https://i0.wp.com/fashionandstylepolice.com/wp-content/uploads/2014/07/square-face-1.jpg?ssl=1r",
    "sepia(40%) hue-rotate(-20deg) brightness(120%)":
      "https://content.latest-hairstyles.com/wp-content/uploads/collarbone-cut-with-soft-waves-1.jpg",
    "contrast(250%) saturate(80%)":
      "https://cdn2.stylecraze.com/wp-content/uploads/2012/06/Most-Flattering-Hairstyles-For-Square-Faces.jpg.webp",
    "hue-rotate(200deg) saturate(150%)":
      "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcS9B4OXhM45p9nYwA-xnU-lO2VaG0rcu-1dhQ&s",
    "invert(100%) hue-rotate(90deg)":
      "https://r2.erweima.ai/imgcompressed/compressed_a08b93c03b426dca04308417d93c5c13.webp",
  };

  return (
    filterImages[filterValue] ||
    "https://via.placeholder.com/150?text=Filter+Not+Found"
  );
}

function switchCamera() {
  // Toggle between front and rear camera
  isFrontCamera.value = !isFrontCamera.value;
  constraints.value.video = {
    facingMode: isFrontCamera.value ? "user" : "environment",
  };
  restartVideoStream();
}

function restartVideoStream() {
  if (video.value && canvas.value) {
    // Stop the current stream if it's running
    const stream = video.value.srcObject;
    const tracks = stream.getTracks();
    tracks.forEach((track) => track.stop());

    // Start the new stream with the updated constraints
    navigator.mediaDevices
      .getUserMedia(constraints.value)
      .then(setStream)
      .catch((e) => {
        console.error(e);
      });
  }
}
</script>

<template>
  <div class="camera-container">
    <!-- Video Element (Hidden) -->
    <video
      ref="video"
      autoplay
      playsinline
      webkit-playsinline
      muted
      hidden
    ></video>

    <!-- Main Camera View -->
    <div class="mt-32 camera-view">
      <canvas ref="canvas" class="main-canvas"></canvas>

      <!-- Camera Overlay UI -->
      <div class="camera-overlay">
        <!-- Top Controls -->
        <div class="top-controls">
          <button class="control-btn settings-btn">
            <i class="fas fa-cog"></i>
          </button>
          <button class="control-btn flash-btn">
            <i class="fas fa-bolt"></i>
          </button>
        </div>

        <!-- Flash Effect -->
        <div v-if="flashVisible" class="flash-effect"></div>

        <!-- Countdown Display -->
        <div v-if="isCountdownActive" class="countdown-display">
          {{ countdown }}
        </div>

        <!-- Bottom Controls -->
        <div class="bottom-controls">
          <!-- Filter Toggle -->
          <button
            @click="filterMenuOpen = !filterMenuOpen"
            class="filter-toggle-btn"
          >
            <i class="fas fa-magic"></i>
          </button>

          <!-- Main Camera Controls -->
          <div class="main-controls">
            <button @click="startCountdown" class="timer-btn">
              <i class="fas fa-clock"></i>
            </button>

            <button @click="takePicture" class="shutter-btn">
              <div class="shutter-inner"></div>
            </button>

            <button @click="switchCamera" class="flip-btn">
              <i class="fas fa-sync-alt"></i>
            </button>
          </div>

          <!-- Gallery Preview -->
          <button class="gallery-btn">
            <div class="preview-thumbnail">
              <img
                src="https://play-lh.googleusercontent.com/Sy_9xv0fpbnc6bJgYM_DGj7-BhleftZgjbeFmvcN5eMGnoizMl2igd5IfJEq82xKZw8"
                alt=""
              />
            </div>
          </button>
        </div>

        <!-- Filter Menu -->
        <div
          v-if="filterMenuOpen"
          class="filter-menu"
          :class="{ 'filter-menu-open': filterMenuOpen }"
        >
          <div class="filter-scroll">
            <button
              v-for="filter in filters"
              :key="filter.value"
              @click="selectedFilter = filter.value"
              class="filter-option"
              :class="{ active: selectedFilter === filter.value }"
            >
              <img :src="getFilterImage(filter.value)" :alt="filter.name" />
              <span>{{ filter.name }}</span>
              <span class="filter-icon">{{ filter.icon }}</span>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import "./Camera.css";
</style>
