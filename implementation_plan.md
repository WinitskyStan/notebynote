# Implementation Plan: NoteByNote - Piano Flashcard App

## 1. Project Overview
A Vue.js application that serves as a musical flashcard system. It allows users to load MIDI files, parses them into sheet music, and displays them bar-by-bar to aid in memorization and practice.

## 2. Technology Stack
-   **Framework:** Vue 3 (Composition API) + Vite
-   **Language:** TypeScript
-   **Styling:** Tailwind CSS (for quick, modern UI)
-   **MIDI Parsing:** `@tonejs/midi` (Converts binary MIDI to JSON)
-   **Music Engraving:** `vexflow` (Standard HTML5 canvas music notation rendering)
-   **State Management:** Vue `ref`/`reactive`

## 3. Core Features & Architecture

### A. MIDI Ingestion
-   **File Upload:** Standard HTML5 file input to accept `.mid` files.
-   **Default Library:** Ability to load the included `wtc-bwv847.mid` by default.
-   **Parsing:** A service that ingests the MIDI ArrayBuffer and converts it to a structured format (Notes, Durations, Pitch) using `@tonejs/midi`.

### B. The "MIDI-to-Score" Engine (The Complex Part)
Since MIDI files do not contain sheet music layout data (stems, clefs, beams), we must interpret the data:
1.  **Quantization:** Round note start times and durations to logical musical units (quarters, eighths, etc.) to avoid messy notation.
2.  **Staff Assignment:** If the MIDI has 2 tracks, assign to Treble/Bass staves. If 1 track, use a "split point" (typically Middle C) to separate hands.
3.  **Bar Formatting:** Calculate measures based on the Time Signature (extracted from MIDI or defaulted to 4/4).

### C. Flashcard Viewer (UI)
-   **Viewport:** Displays a specific range of measures (e.g., Measure 5 to 6).
-   **Controls:**
    -   *Navigation:* Next/Previous buttons.
    -   *Visibility:* "Hide Music" toggle (blanks the staves).
    -   *Settings:* "Bars per Page" (Input/Slider), "Start Hidden" (Checkbox).

### D. Data Flow
1.  User selects file -> `MidiParser` converts to `SongObject`.
2.  `SongObject` contains a list of `Measures`.
3.  `ScoreRenderer` component accepts `Measures` + `CurrentIndex` + `VisibilityState`.
4.  VexFlow draws the specific measures on the `<canvas>`.

## 4. Development Phases

### Phase 1: Setup & Infrastructure
-   Initialize Vite + Vue + TS project.
-   Install dependencies (`vexflow`, `@tonejs/midi`, `tailwindcss`).
-   Set up basic layout.

### Phase 2: MIDI Parsing Logic
-   Implement the logic to read `wtc-bwv847.mid`.
-   Log parsed notes to console to verify structure.
-   Implement basic Quantization (converting absolute time to musical duration).

### Phase 3: Rendering Engine
-   Create a `ScoreDisplay` component.
-   Use VexFlow to draw a static C Major scale (proof of concept).
-   Connect parsed MIDI data to VexFlow staves.
-   Implement the "Bar Slicing" logic (only render specific measures).

### Phase 4: Flashcard Interactivity
-   Add "Next/Prev" buttons.
-   Implement the "Hide/Mask" overlay feature.
-   Add configuration inputs (Bars count, Auto-hide).

### Phase 5: Polish & Refine
-   Improve CSS styling.
-   Handle edge cases (empty bars, complex rhythms).
-   Verify with the provided `wtc-bwv847.mid`.

## 5. Directory Structure
```
src/
  components/
    Controls.vue       # Buttons and settings
    ScoreDisplay.vue   # VexFlow canvas wrapper
    FileSelector.vue   # Upload or select default
  composables/
    useMidiParser.ts   # Logic to convert MIDI -> VexFlow data
  App.vue
  main.ts
```
