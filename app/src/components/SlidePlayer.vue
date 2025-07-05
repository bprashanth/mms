<template>
    <div v-if="currentSlide" class="slide-container">
        <div class="controls">
            <button @click="toggleAudio">
                {{ isPlaying ? 'Pause' : 'Play' }}
            </button>
        </div>
        <div v-if="currentSlide.asset" class="asset-wrapper">
            <img 
            v-if="isImage(currentSlide.asset)" 
            :src="currentSlide.asset" alt="slide asset" 
            class="asset-image" />

            <video 
            v-else 
            :src="currentSlide.asset" 
            controls class="asset-video"/>
        </div>

        <p class="slide-text">{{ currentSlide.text }}</p>

        <audio :src="audioSrc" ref="audioRef" preload="auto"></audio>
    </div>
    <div v-else class="loading">Loading...</div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const slides = ref([])
const currentSlideIndex = ref(0)
const currentSlide = ref(null)
const audioRef = ref(null)
const isPlaying = ref(false)
const audioSrc = '/audio/web_intro.mp3'

const isImage = (path) => {
    console.log('Checking if path is an image: ', path)
    return path.match(/\.(jpeg|jpg|gif|png|webp)$/i)
}

onMounted(async () => {
    const rest = await fetch('/html_nature.json')
    const data = await rest.json()
    slides.value = data 
    currentSlide.value = slides.value[currentSlideIndex.value]
})

const toggleAudio = () => {
    if (!audioRef.value) return 
    if (isPlaying.value) {
        audioRef.value.pause()
    } else {
        audioRef.value.play()
    }
    isPlaying.value = !isPlaying.value
}
</script>

<style scoped>
.slide-container {
    background-color: black;
    color: white;
    font-family: 'Arial', sans-serif;
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 2rem;
}

.slide-text {
    font-size: 1.5rem;
    line-height: 1.6;
    margin-top: 1rem;
}

.asset-wrapper {
    max-width: 80%;
    margin-bottom: 2rem;
}

.asset-image,
.asset-video {
    max-width: 100%;
    max-height: 60vh;
}
.controls {
    position: absolute;
    top: 1rem;
    right: 1rem;
}

button {
    background: #ffffff20;
    color: white;
    border: 1px solid white;
    padding: 0.5rem 1rem;
    font-size: 1rem;
    cursor: pointer;
    border-radius: 5px;
    transition: background 0.3s ease;
}

button:hover {
    background: #ffffff30;
}
</style>