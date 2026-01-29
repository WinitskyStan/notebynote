# Notecard - Piano Flashcard App

Notecard is a minimalist, elegant web application designed to help pianists memorize music bar-by-bar. It transforms complex MusicXML scores into manageable "cards," allowing for focused practice and memorization.

## 🎨 Design Philosophy
The application follows a "classic paper" aesthetic, inspired by traditional physical flashcards and high-quality sheet music printing.

-   **Typography:** Combines **Playfair Display** (Serif) for an elegant, musical feel with **Roboto Mono** for technical metadata.
-   **Color Palette:** A warm "Cream" background (`#F9F8F6`) with "Earthy Brown" accents (`#6D4C41`).
-   **UI:** A centered "Card" container with subtle dot-grid backgrounds, emphasizing focus and clarity.

## 🛠 Technology Stack
-   **Framework:** Vue 3 (Composition API) + Vite
-   **Language:** TypeScript
-   **Styling:** Tailwind CSS 4
-   **Sheet Music Engine:** [OpenSheetMusicDisplay (OSMD)](https://opensheetmusicdisplay.org/)
-   **Icons:** Lucide-Vue-Next
-   **Fonts:** @fontsource (Playfair Display, Roboto Mono, Inter)

## 🏗 Project Structure
```text
src/
├── components/
│   └── ScoreDisplay.vue   # Core OSMD wrapper; handles zooming, centering, and hidden states.
├── utils/                 # (Cleaned up) Formerly held MIDI logic; now relies on OSMD internal parsing.
├── App.vue                # Main layout, state management (bars per card, navigation), and keyboard listeners.
├── main.ts                # Application entry point.
└── style.css              # Custom theme definitions and global dot-grid background.

public/
└── wtc-bwv847.mxl         # Default MusicXML score (Bach Prelude No. 2 in C Minor).
```

## 🚀 Key Features

### 1. Progressive Memorization
-   **Flashcard Mode:** Music is hidden by default. Use the central "Eye" button or `Spacebar` to reveal.
-   **Customizable Range:** Set the "Bars per card" (1-4) via the Preferences menu or keyboard shortcuts `1`-`4`.
-   **Auto-hide:** Optional setting to automatically hide the next measure when navigating, forcing a memory recall.

### 2. Navigation & Controls
-   **Smooth Stepping:** Navigate measures using `Left`/`Right` arrows or UI buttons.
-   **Progress Tracking:** A minimalist progress bar at the bottom indicates your position in the entire piece.
-   **Smart Indicators:** A dynamic header (e.g., "Measures 2 – 3") and a large background watermark indicate the current bar number.

### 3. Keyboard Shortcuts
| Key | Action |
| :--- | :--- |
| `Space` | Reveal / Hide Music |
| `Right Arrow` | Next Measure(s) |
| `Left Arrow` | Previous Measure(s) |
| `1` - `4` | Set Bars per Card |

## 📦 Getting Started

### Installation
```bash
npm install
```

### Development
```bash
npm run dev
```

### Build
```bash
npm run build
```

---
*Created with a focus on minimalist design and functional practice.*