<template>
    <div v-if="currentSlide" class="slide-container">
        <div class="controls">
            <button @click="toggleAudio">
                {{ isPlaying ? 'Pause' : 'Play' }}
            </button>
            <div class="progress-container">
                <span class="time-display">{{ formatTime(progress) }}</span>
                <input 
                type="range" 
                min="0" 
                :max="duration" 
                step="0.01" 
                v-model="progress" 
                @input="onSeek" 
                @mousedown="onSeekStart"
                @mouseup="onSeekEnd"
                />
                <span class="time-display">{{ formatTime(duration) }}</span>
            </div>
        </div>
        <div v-if="currentSlide.asset" class="asset-wrapper">
            <img 
            v-if="isImage(currentSlide.asset)" 
            :src="currentSlide.asset" alt="slide asset" 
            class="asset-image" />

            <video 
            v-else 
            :src="currentSlide.asset" 
            :autoplay="isPlaying"
            muted
            loop
            class="asset-video"
            ref="videoRef"/>
        </div>

        <p :class="['slide-text', currentSlide.animation || '']">{{ currentSlide.text }}</p>

        <audio :src="audioSrc" ref="audioRef" preload="auto"></audio>
    </div>
    <div v-else class="loading">Loading...</div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue';

const slides = ref([])
const currentSlideIndex = ref(0)
const currentSlide = ref(null)
const audioRef = ref(null)
const videoRef = ref(null)
const isPlaying = ref(false)
const audioSrc = '/audio/nature_web.mp3'

// Progress bar for audio 
const progress = ref(0)
const duration = ref(1)
const isSeeking = ref(false)

onMounted(async () => {
    const rest = await fetch('/transcript.json')
    const data = await rest.json()
    slides.value = data.map(
        d => ({
            start: d.start ?? (d.timestamp ? d.timestamp[0] : 0),
            end: d.end ?? (d.timestamp ? d.timestamp[1] : 0),
            text: d.displayText,
            fullText: d.fullText,
            asset: d.asset ?? null, 
            animation: d.animation ?? null
        })
    )
    currentSlide.value = slides.value[currentSlideIndex.value]

    // Set up audio event listeners
    await nextTick()
    if (audioRef.value) {
        audioRef.value.addEventListener('loadedmetadata', () => {
            duration.value = audioRef.value.duration
        })
        
        audioRef.value.addEventListener('timeupdate', () => {
            if (audioRef.value && !isSeeking.value) {
                progress.value = audioRef.value.currentTime
            }
        })
    }

    setInterval(() => {
        if (!audioRef.value || !isPlaying.value) return 
        const currentTime = audioRef.value.currentTime 
        
        // Update progress (but only if not currently seeking)
        if (!isSeeking.value) {
            progress.value = currentTime
        }
        
        const index = slides.value.findIndex(
            slide => currentTime >= slide.start && currentTime < slide.end)
        if (index !== -1 && index !== currentSlideIndex.value) {
            currentSlideIndex.value = index
            currentSlide.value = slides.value[index]
        }
    }, 100)
})

const toggleAudio = () => {
    if (!audioRef.value) return 
    if (isPlaying.value) {
        audioRef.value.pause()
        if (videoRef.value) {
            videoRef.value.pause()
        }
    } else {
        audioRef.value.play()
        if (videoRef.value) {
            videoRef.value.play()
        }
    }
    isPlaying.value = !isPlaying.value
}

const isImage = (path) => {
    console.log('Checking if path is an image: ', path)
    return path.match(/\.(jpeg|jpg|gif|png|webp)$/i)
}

const onSeek = () => {
    if (audioRef.value) {
        audioRef.value.currentTime = parseFloat(progress.value)
    }
}

const onSeekStart = () => {
    isSeeking.value = true
}

const onSeekEnd = () => {
    isSeeking.value = false
}

const formatTime = (seconds) => {
    if (!seconds || isNaN(seconds)) return '0:00'
    const mins = Math.floor(seconds / 60)
    const secs = Math.floor(seconds % 60)
    return `${mins}:${secs.toString().padStart(2, '0')}`
}
</script>

<style scoped>
.slide-container {
    background-color: black;
    color: white;
    font-family: 'Arial', sans-serif;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 2rem;
    width: 100%;
}

.slide-text {
    font-size: 1.5rem;
    line-height: 1.6;
    margin-top: 1rem;
    opacity: 1;
    transition: opacity 0.5s ease; 
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

.progress-container {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    margin-top: 0.5rem;
}

.time-display {
    font-size: 0.8rem;
    color: white;
    min-width: 2.5rem;
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

input[type="range"] {
    width: 150px;
    accent-color: white;
}

/* Text animations */
.fade-in {
    animation: fadeIn 5s ease forwards;
}

.slide-in {
    animation: slideIn 0.8s ease forwards;
}

.typewriter {
    overflow: hidden;
    border-right: 0.15em solid white;
    white-space: nowrap;
    animation: typing 5s steps(40, end), blink-caret 1s step-end infinite; 
}

.pulse {
    animation: pulseAnim 3s infinite;
}
@keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
}
@keyframes slideId {
    from { opacity: 0; transform: translateY(20px); }
    to { opacity: 1; transform: translateY(0); }
}
@keyframes typing {
    from { width: 0; }
    to { width: 100%; }
}
@keyframes blink-caret {
    from, to { border-color: transparent; }
    50% { border-color: white; }
}
@keyframes pulseAnim {
    0% { transform: scale(1); opacity: 1; }
    50% { transform: scale(1.05); opacity: 0.7; }
    100% { transform: scale(1); opacity: 1; } 
}
</style>