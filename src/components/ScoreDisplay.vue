<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch } from 'vue';
import { OpenSheetMusicDisplay } from 'opensheetmusicdisplay';
import { EyeOff } from 'lucide-vue-next';

const props = defineProps<{
  url: string;
  startMeasure: number;
  measureCount: number;
  hideMusic: boolean;
  showNoteNames: boolean;
}>();

const containerRef = ref<HTMLElement | null>(null);
let osmd: OpenSheetMusicDisplay | null = null;
const isReady = ref(false);
const totalMeasures = ref(0);

const emit = defineEmits(['loaded']);

const calculateLabelPosition = (graphicalNote: any) => {
  if (graphicalNote.PositionAndShape && graphicalNote.PositionAndShape.AbsolutePosition) {
    const absPos = graphicalNote.PositionAndShape.AbsolutePosition;
    const stemDirection = graphicalNote.sourceNote.StemDirection;
    
    // Move labels closer to the note head
    // StemDirection: 1 = Up (label below?), 2 = Down (label above?)
    // Actually, usually if stem is up, note head is at bottom, so label below?
    // Or if stem is up, we can put label above the stem?
    // Let's try to put them consistently relative to the note head.
    // If stem is UP (1), note head is at the bottom of the stem.
    // If stem is DOWN (2), note head is at the top of the stem.
    
    // OSMD AbsolutePosition for GraphicalNote is usually the note head position.
    const isMobile = window.innerWidth < 768;
    const yOffset = stemDirection === 1 
      ? (isMobile ? 2.0 : 2.5) 
      : (isMobile ? -3.0 : -3.5); 

    return {
      x: absPos.x - 0.5, // Center it slightly better
      y: absPos.y + yOffset,
      stemDirection
    };
  }
  
  return { x: 0, y: 0, stemDirection: 0 };
};

const createLabelElement = (pitchName: string, position: {x: number, y: number}) => {
  const container = containerRef.value;
  if (!container) return;

  const label = document.createElement('div');
  label.className = 'note-name-label';
  label.textContent = pitchName;
  label.style.position = 'absolute';
  
  const zoom = osmd?.Zoom || 1;
  const unitInPixels = 10;
  
  label.style.left = `${position.x * unitInPixels * zoom}px`; 
  label.style.top = `${position.y * unitInPixels * zoom}px`;
  
  // Centering (width/margin-left handled here to maintain coordinate system logic)
  label.style.width = '40px';
  label.style.textAlign = 'center';
  label.style.marginLeft = '-20px'; 

  // Create or use a labels layer
  let labelsLayer = container.querySelector('.note-labels-layer') as HTMLElement;
  if (!labelsLayer) {
    labelsLayer = document.createElement('div');
    labelsLayer.className = 'note-labels-layer';
    labelsLayer.style.position = 'absolute';
    labelsLayer.style.top = '0';
    labelsLayer.style.left = '0';
    labelsLayer.style.width = '100%';
    labelsLayer.style.height = '100%';
    labelsLayer.style.pointerEvents = 'none';
    container.appendChild(labelsLayer);
  }

  labelsLayer.appendChild(label);
};

const clearLabels = () => {
  console.log("clear labels") ;
  const container = containerRef.value;
  if (!container) return;

  const labelsLayer = container.querySelector('.note-labels-layer');
  if (labelsLayer) {
    labelsLayer.remove();
  }
};

const addNoteLabels = () => {
  if (!osmd || !osmd.GraphicSheet) return;

  // Traverse OSMD structure
  osmd.GraphicSheet.MeasureList.forEach((measureList: any) => {
    measureList.forEach((measure: any) => {
      const number = measure.measureNumber;
      // Allow if within range OR if it's the pre-measure (0) and we are at the start (1)
      const isInRange = (number >= props.startMeasure && number < props.startMeasure + props.measureCount);
      const isPreMeasure = (props.startMeasure === 1 && number === 0);

      if (!isInRange && !isPreMeasure) return;

      measure.staffEntries.forEach((staffEntry: any) => {
        staffEntry.graphicalVoiceEntries.forEach((graphicalVoiceEntry: any) => {
          graphicalVoiceEntry.notes.forEach((graphicalNote: any) => {
            //if (!graphicalNote.isVisible) return;

            const pitch = graphicalNote.sourceNote.Pitch;
            if (!pitch) return;

            // Use ToStringShort for clean output like "C4", "Bb5"
            const pitchName = pitch.ToStringShort ? pitch.ToStringShort() : pitch.ToString();

            // Get position for label
            const position = calculateLabelPosition(graphicalNote);

            // Create and position label
            if (position) {
              createLabelElement(pitchName, position);
            }
          });
        });
      });
    });
  });
};

const loadScore = async () => {
  if (!containerRef.value) return;

  if (!osmd) {
    osmd = new OpenSheetMusicDisplay(containerRef.value, {
      autoResize: false, // Handle manually to sync labels
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
  
  // Dynamic zoom based on screen width
  const isMobile = window.innerWidth < 768;
  osmd.Zoom = isMobile ? 0.7 : 1.0;

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

      // Render note names if enabled
      clearLabels();
      if (props.showNoteNames) {
        addNoteLabels();
      }

  } catch (e) {
      console.error("Render Error:", e);
  }
};

watch(() => props.url, loadScore);
watch(() => [props.startMeasure, props.measureCount, props.showNoteNames], render);

const handleResize = () => {
  render();
};

onMounted(() => {
  loadScore();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
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
  </div>
</template>

<style scoped>
:deep(svg) {
    max-width: 100%;
    max-height: 100%;
}
</style>