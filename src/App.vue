<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { 
  Music, 
  Settings2, 
  Upload, 
  ChevronLeft, 
  ChevronRight, 
  Eye, 
  EyeOff,
  Keyboard
} from 'lucide-vue-next';
import ScoreDisplay from './components/ScoreDisplay.vue';

// State
const musicUrl = ref('./chopin-nocturne-op-9-no-1-b-flat-minor.mxl');
const currentMeasure = ref(1);
const barsPerPage = ref(1);
const totalMeasures = ref(0);
const isMusicHidden = ref(false); // Default to visible on load
const showSettings = ref(false);
const autoHideOnNext = ref(false);
const showNoteNames = ref(false);

// Touch state for swiping
const touchStartX = ref(0);
const touchEndX = ref(0);
const minSwipeDistance = 50;

const handleTouchStart = (e: TouchEvent) => {
  const touch = e.changedTouches[0];
  if (touch) {
    touchStartX.value = touch.screenX;
  }
};

const handleTouchEnd = (e: TouchEvent) => {
  const touch = e.changedTouches[0];
  if (touch) {
    touchEndX.value = touch.screenX;
    handleSwipe();
  }
};

const handleSwipe = () => {
  const distance = touchEndX.value - touchStartX.value;
  if (Math.abs(distance) < minSwipeDistance) return;

  if (distance > 0) {
    prev();
  } else {
    next();
  }
};

const progress = computed(() => {
  if (totalMeasures.value === 0) return 0;
  return (currentMeasure.value / totalMeasures.value) * 100;
});

const handleLoaded = (total: number) => {
  totalMeasures.value = total;
};

const next = () => {
  if (currentMeasure.value + barsPerPage.value <= totalMeasures.value) {
    currentMeasure.value += barsPerPage.value;
    isMusicHidden.value = autoHideOnNext.value;
  }
};

const prev = () => {
  if (currentMeasure.value - barsPerPage.value >= 1) {
    currentMeasure.value -= barsPerPage.value;
    isMusicHidden.value = autoHideOnNext.value;
  } else {
    currentMeasure.value = 1;
    isMusicHidden.value = autoHideOnNext.value;
  }
};

const goToStart = () => {
  currentMeasure.value = 1;
  isMusicHidden.value = autoHideOnNext.value;
};

const goToEnd = () => {
  const lastMeasure = Math.max(1, totalMeasures.value - barsPerPage.value + 1);
  currentMeasure.value = lastMeasure;
  isMusicHidden.value = autoHideOnNext.value;
};

const toggleReveal = () => {
  isMusicHidden.value = !isMusicHidden.value;
};

const closeSettings = () => {
  showSettings.value = false;
};

// Keyboard Listeners
const handleKeydown = (e: KeyboardEvent) => {
  if (e.code === 'Space') {
    e.preventDefault();
    toggleReveal();
  } else if (e.code === 'ArrowRight') {
    next();
  } else if (e.code === 'ArrowLeft') {
    prev();
  } else if (e.code === 'Home') {
    e.preventDefault();
    goToStart();
  } else if (e.code === 'End') {
    e.preventDefault();
    goToEnd();
  } else if (e.code === 'ArrowUp') {
    e.preventDefault();
    showNoteNames.value = !showNoteNames.value;
  } else if (['1', '2', '3', '4'].includes(e.key)) {
    barsPerPage.value = parseInt(e.key);
  }
};

onMounted(() => {
  window.addEventListener('keydown', handleKeydown);
  window.addEventListener('click', closeSettings);
});

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
  window.removeEventListener('click', closeSettings);
});

const handleFileSelect = () => {
  alert("TODO: Implement File Upload (MusicXML)");
};
</script>

<template>
  <div 
    class="min-h-screen flex flex-col items-center"
    @touchstart="handleTouchStart"
    @touchend="handleTouchEnd"
  >
    
    <!-- Header -->
    <header class="w-full max-w-6xl px-8 py-6 flex justify-between items-center relative z-40">
      <div class="flex items-center gap-3">
        <div class="p-2 bg-white rounded-full shadow-sm border border-gray-100 text-brand-brown">
          <Music class="w-6 h-6" />
        </div>
        <span class="font-serif text-3xl tracking-tight text-brand-dark select-none">Notecard</span>
      </div>

      <div class="flex items-center gap-4">
        <button 
            @click.stop="showSettings = !showSettings"
            class="p-3 text-gray-400 hover:text-brand-brown hover:bg-white/50 rounded-full transition-all active:scale-90"
            title="Settings"
        >
          <Settings2 class="w-6 h-6" />
        </button>
        <button 
          @click="handleFileSelect"
          class="flex items-center gap-2 px-4 py-2 border border-gray-200 rounded-lg text-sm font-medium text-gray-600 hover:bg-white hover:shadow-sm transition-all"
        >
          <Upload class="w-4 h-4" />
          Upload MIDI
        </button>
      </div>
    </header>

    <!-- Settings Dropdown (Floating) -->
    <div 
        v-if="showSettings" 
        @click.stop
        class="fixed sm:absolute top-24 left-1/2 -translate-x-1/2 sm:left-auto sm:right-8 sm:translate-x-0 z-[100] w-[calc(100%-2rem)] sm:w-72 bg-white rounded-2xl shadow-2xl border border-gray-100 p-6 animate-in fade-in slide-in-from-top-4 duration-300"
    >
        <h3 class="font-serif text-xl text-brand-brown mb-4">Preferences</h3>
        <p class="text-xs text-gray-400 mb-6">Customize your practice session.</p>
        
        <div class="space-y-6">
            <div class="flex items-center justify-between gap-4">
                <span class="text-sm font-medium text-gray-700">Auto-hide on next</span>
                <div class="flex p-1 bg-gray-100 rounded-lg shrink-0">
                    <button 
                        @click="autoHideOnNext = false" 
                        class="px-3 py-1 rounded-md text-[10px] font-bold transition-all font-['Oswald']"
                        :class="!autoHideOnNext ? 'bg-white shadow-sm text-brand-dark' : 'text-gray-400'"
                    >OFF</button>
                    <button 
                        @click="autoHideOnNext = true" 
                        class="px-3 py-1 rounded-md text-[10px] font-bold transition-all font-['Oswald']"
                        :class="autoHideOnNext ? 'bg-brand-brown text-white shadow-sm' : 'text-gray-400'"
                    >ON</button>
                </div>
            </div>

            <div class="flex items-center justify-between gap-4">
                <span class="text-sm font-medium text-gray-700">Show note names</span>
                <div class="flex p-1 bg-gray-100 rounded-lg shrink-0">
                    <button 
                        @click="showNoteNames = false" 
                        class="px-3 py-1 rounded-md text-[10px] font-bold transition-all font-['Oswald']"
                        :class="!showNoteNames ? 'bg-white shadow-sm text-brand-dark' : 'text-gray-400'"
                    >OFF</button>
                    <button 
                        @click="showNoteNames = true" 
                        class="px-3 py-1 rounded-md text-[10px] font-bold transition-all font-['Oswald']"
                        :class="showNoteNames ? 'bg-brand-brown text-white shadow-sm' : 'text-gray-400'"
                    >ON</button>
                </div>
            </div>

            <div>
                <div class="flex items-center justify-between mb-2">
                    <span class="text-sm font-medium text-gray-700">Bars per card</span>
                    <span class="text-xs font-mono bg-gray-100 px-2 py-0.5 rounded text-gray-500">{{ barsPerPage }}</span>
                </div>
                <input type="range" v-model.number="barsPerPage" min="1" max="4" class="w-full accent-brand-brown">
                <p class="text-[10px] text-gray-400 mt-2 italic">Show more context by displaying multiple bars at once.</p>
            </div>
        </div>
    </div>

    <!-- Main Card -->
    <main class="flex-1 w-full max-w-6xl px-8 flex flex-col items-center justify-center -mt-12">
        <!-- Measure Indicator -->
        <div class="mb-6 flex flex-col items-center">
            <span class="font-serif text-2xl text-brand-brown">
                {{ barsPerPage === 1 ? `Measure ${currentMeasure} of ${totalMeasures}` : `Measures ${currentMeasure} – ${Math.min(currentMeasure + barsPerPage - 1, totalMeasures)} of ${totalMeasures}` }}
            </span>
            <div class="h-0.5 w-12 bg-brand-brown/20 mt-2 rounded-full"></div>
        </div>

        <div 
            class="w-full bg-white rounded-[2rem] shadow-xl border border-gray-100 overflow-hidden relative group"
        >
            <ScoreDisplay 
                :url="musicUrl" 
                :startMeasure="currentMeasure" 
                :measureCount="barsPerPage"
                :hideMusic="isMusicHidden"
                :showNoteNames="showNoteNames" 
                @loaded="handleLoaded"
            />
        </div>

        <!-- Progress Bar -->
        <div class="w-full max-w-3xl h-1 bg-gray-200/50 rounded-full mt-12 overflow-hidden">
            <div class="h-full bg-brand-brown transition-all duration-500" :style="{ width: `${progress}%` }"></div>
        </div>

        <!-- Main Controls -->
        <div class="mt-8 flex items-center gap-6">
            <button 
                @click="prev"
                :disabled="currentMeasure <= 1"
                class="w-14 h-14 flex items-center justify-center rounded-full border border-gray-100 bg-white shadow-sm text-gray-400 hover:text-brand-brown hover:shadow-md disabled:opacity-30 disabled:hover:shadow-sm transition-all"
            >
                <ChevronLeft class="w-6 h-6" />
            </button>

            <button 
                @click="toggleReveal"
                class="w-20 h-20 flex items-center justify-center rounded-full bg-brand-brown text-white shadow-lg hover:shadow-2xl hover:scale-105 active:scale-95 transition-all"
            >
                <component :is="isMusicHidden ? Eye : EyeOff" class="w-8 h-8" />
            </button>

            <button 
                @click="next"
                :disabled="currentMeasure + barsPerPage > totalMeasures + 1"
                class="w-14 h-14 flex items-center justify-center rounded-full border border-gray-100 bg-white shadow-sm text-gray-400 hover:text-brand-brown hover:shadow-md disabled:opacity-30 disabled:hover:shadow-sm transition-all"
            >
                <ChevronRight class="w-6 h-6" />
            </button>
        </div>

        <!-- Footer Shortcuts -->
        <div class="mt-8 flex items-center gap-8 text-[11px] font-medium text-gray-400 uppercase tracking-widest">
            <div class="flex items-center gap-2">
                <div class="p-1 border border-gray-200 rounded shadow-xs bg-white">
                    <Keyboard class="w-3 h-3" />
                </div>
                <span><span class="text-gray-500 font-bold">Space</span> Reveal</span>
            </div>
            <div class="flex items-center gap-2">
                <div class="p-1 border border-gray-200 rounded shadow-xs bg-white min-w-[24px] text-center font-bold text-gray-500">
                    &uarr;
                </div>
                <span>Note Names</span>
            </div>
            <div class="flex items-center gap-2">
                <div class="p-1 border border-gray-200 rounded shadow-xs bg-white">
                    &larr;/&rarr;
                </div>
                <span>Navigate</span>
            </div>
        </div>
    </main>
  </div>
</template>

<style>
input[type="range"] {
  -webkit-appearance: none;
  background: transparent;
}

input[type="range"]::-webkit-slider-runnable-track {
  width: 100%;
  height: 4px;
  background: #f3f4f6;
  border-radius: 2px;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #6d4c41;
  margin-top: -6px;
  cursor: pointer;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}
</style>

<style>
/* Global styles if needed */
</style>