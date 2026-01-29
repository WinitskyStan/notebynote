<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import { OpenSheetMusicDisplay } from 'opensheetmusicdisplay';
import { EyeOff } from 'lucide-vue-next';

const props = defineProps<{
  url: string;
  startMeasure: number;
  measureCount: number;
  hideMusic: boolean;
}>();

const containerRef = ref<HTMLElement | null>(null);
let osmd: OpenSheetMusicDisplay | null = null;
const isReady = ref(false);
const totalMeasures = ref(0);

const emit = defineEmits(['loaded']);

const loadScore = async () => {
  if (!containerRef.value) return;

  if (!osmd) {
    osmd = new OpenSheetMusicDisplay(containerRef.value, {
      autoResize: true,
      backend: 'svg',
      drawingParameters: 'compacttight',
      drawPartNames: false,
      drawTitle: false,
      drawSubtitle: false,
      drawComposer: false,
      drawLyricist: false,
      renderSingleHorizontalStaffline: false,
      stretchLastSystemLine: true, // Prevents stretching the measure to full container width
    });
  }

  try {
    await osmd.load(props.url);
    totalMeasures.value = osmd.Sheet.SourceMeasures.length;
    emit('loaded', totalMeasures.value);
    isReady.value = true;
    render();
  } catch (e) {
    console.error("OSMD Load Error:", e);
  }
};

const render = () => {
  if (!osmd || !isReady.value) return;

  const endMeasure = props.startMeasure + props.measureCount - 1;
  
  // Set a good default zoom
  osmd.Zoom = 1.4;

  osmd.setOptions({
    drawFromMeasureNumber: props.startMeasure,
    drawUpToMeasureNumber: Math.min(endMeasure, totalMeasures.value)
  });

  try {
      osmd.render();
      // Apply subtle styling to SVG elements if needed
      const svg = containerRef.value?.querySelector('svg');
      if (svg) {
          svg.style.filter = 'contrast(0.9) brightness(1.1)'; // Slightly softer black
      }
  } catch (e) {
      console.error("Render Error:", e);
  }
};

watch(() => props.url, loadScore);
watch(() => [props.startMeasure, props.measureCount], render);

onMounted(() => {
  loadScore();
});
</script>

<template>
  <div class="relative w-full flex items-center justify-center p-8">
    
    <!-- The Score Container -->
    <div 
        ref="containerRef" 
        class="w-full h-full flex items-center justify-center transition-opacity duration-500"
        :class="hideMusic ? 'opacity-0 scale-95' : 'opacity-100 scale-100'"
    ></div>

    <!-- Hidden State Overlay -->
    <div 
        v-if="hideMusic" 
        class="absolute inset-0 flex flex-col items-center justify-center text-gray-400 gap-3 animate-in fade-in zoom-in duration-300"
    >
        <div class="p-8 rounded-full bg-white shadow-sm border border-gray-100">
            <EyeOff class="w-12 h-12 stroke-[1.5]" />
        </div>
        <span class="font-serif italic text-xl">Hidden</span>
    </div>

    <!-- Background Loading -->
    <div v-if="!isReady" class="absolute inset-0 flex items-center justify-center bg-white/50">
        <div class="w-8 h-8 border-4 border-brand-brown/20 border-t-brand-brown rounded-full animate-spin"></div>
    </div>

    <!-- Measure Number Indicator -->
    <div class="absolute bottom-6 right-8 text-gray-100 font-serif text-8xl pointer-events-none select-none">
        {{ startMeasure }}
    </div>
  </div>
</template>

<style scoped>
:deep(svg) {
    max-width: 100%;
    max-height: 100%;
}
</style>