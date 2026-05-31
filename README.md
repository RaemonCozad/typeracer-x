# ⌨️ TYPERACER X

A competitive typing speed game with an adaptive AI opponent, ghost replay system, live WPM tracking, combo mechanics, and a full stats dashboard — built entirely in vanilla HTML, CSS, and JavaScript.

🔗 **[Play it live →](https://raemoncozad.github.io/typeracer-x/)**

---

## 📸 Preview

<img width="1132" height="939" alt="TypeRacerXAnimation" src="https://github.com/user-attachments/assets/cafbd68c-ec08-44fd-841c-9fcd9f8ca870" />

---

## 🎮 How to Play

Type the displayed quote as fast and accurately as you can before the AI finishes it. Every race is saved — your stats improve over time and the AI adapts to match your skill level.

| Key | Action |
|---|---|
| Just start typing | Starts the timer on first keystroke |
| `Space` | Quick restart (outside of a race) |
| `Tab` | Load a new random quote |

**Scoring:**
- Build a combo by typing consecutive correct characters
- Every 20-combo milestone triggers a bonus flash and rising audio tone
- One wrong character resets your combo to zero
- Final WPM, accuracy, max combo, and time are shown at the end of each race

---

## ✨ Features

### Race Mechanics
- 🤖 **Adaptive AI opponent** — reads your personal best WPM and blends it with the chosen difficulty, so the AI always feels like a real challenge rather than a fixed target
- 👻 **Ghost replay** — your best-ever run is saved as a timestamped progress log; your ghost races alongside you in every subsequent race at the exact pace of your record
- 🏁 **Live race track** — animated progress bars for You, AI, and Ghost update every frame with current WPM displayed alongside each racer
- 💥 **Combo system** — consecutive correct characters build a streak; milestones trigger a fullscreen flash animation and a synthesised chime that rises in pitch

### Typing Engine
- ✅ **Per-character colour feedback** — correct chars turn green, errors turn red with a background highlight, untyped chars stay dimmed
- 📍 **Animated cursor** — a glowing underline tracks your current position in the quote
- 🔴 **Error border** — the input field flashes red when a mistyped word is detected
- 📊 **Standard WPM formula** — `(characters typed / 5) / minutes elapsed`, recalculated every frame — not an estimate

### Stats & Persistence
- 📈 **WPM history chart** — hand-drawn on Canvas; shows your last 20 races as a gradient line chart with grid, dots, and live redraws on window resize
- 🏆 **Personal bests** — best WPM, best accuracy, best combo, win rate, and race count all persist across sessions via `localStorage`
- 👻 **Ghost persistence** — your best run's timestamped progress is stored and replayed as the ghost racer in every future race
- 🎖️ **New PB banner** — a special golden badge and ascending 5-note fanfare plays when you break your WPM record

### Audio (Zero External Files)
- 🔊 All sound effects synthesised in real time using the **Web Audio API**
- Distinct tones for: keypress click, error buzz, combo chime (pitch-scaled by streak), victory fanfare, defeat sting, and personal best melody

### UI & Design
- 🌑 Deep dark theme with cyan/orange accent palette and scanline overlay texture
- `Bebas Neue` display font + `DM Mono` monospace for stats and typing — zero generic font choices
- CSS Grid layout with a fixed sidebar — everything visible at once, nothing hidden behind tabs
- 4 difficulty levels: Easy, Medium, Hard, Insane — each with different base WPM, variance, and adaptation weight

---

## 🛠️ Tech Stack

| Technology | Usage |
|---|---|
| HTML5 Canvas | WPM history chart — drawn entirely with the 2D API |
| CSS Grid & Variables | Layout, theming, and dark mode |
| Vanilla JavaScript | All game logic, state, AI, and input handling |
| `requestAnimationFrame` | 60fps game loop for race simulation |
| Web Audio API | Procedurally synthesised sound effects |
| `localStorage` | Full stats and ghost replay persistence |

**No frameworks. No libraries. No build step.**

---

## 🚀 Run Locally

```bash
git clone https://github.com/yourusername/typeracer-x.git
open index.html
```

No install. No server required.

---

## 🧠 Technical Deep Dives

### Adaptive AI Algorithm

The AI's target WPM is not a fixed number — it's a weighted blend of the player's personal best and the difficulty preset, so it always scales to your current skill level:

```js
const adaptedWpm = playerBestWpm > 0
  ? playerBestWpm * difficulty.adapt + difficulty.baseWpm * (1 - difficulty.adapt)
  : difficulty.baseWpm;
```

On Medium (`adapt: 0.5`), if your best is 80 WPM the AI targets ~67 WPM — competitive but beatable. On Insane (`adapt: 0.9`) it tracks 90% of your best, making it nearly impossible to outrun yourself.

During the race, the AI's current WPM also fluctuates with randomised variance every few seconds, lerped smoothly to avoid jarring jumps:

```js
G.aiCurrentWpm += (G.aiWpmTarget - G.aiCurrentWpm) * 0.05; // smooth lerp
```

---

### Ghost Replay System

Every time you type, your current progress ratio (0–1) and elapsed time are pushed to a log array:

```js
G.progressLog.push({ t: elapsedSeconds, prog: charsTyped / totalChars });
```

When you set a new personal best, this log is persisted to `localStorage`. In the next race, the ghost racer replays it by scanning the array for entries whose timestamp has passed:

```js
while (ghostIdx < ghostLog.length && ghostLog[ghostIdx].t <= elapsed) {
  ghostProgress = ghostLog[ghostIdx].prog;
  ghostIdx++;
}
```

No interpolation needed — the position updates at the same cadence the original run was recorded, making it an exact replay.

---

### WPM Calculation

WPM is calculated using the typing industry standard — every 5 characters counts as one "word", regardless of actual word length:

```js
const words = charsCorrectlyTyped / 5;
const minutes = elapsedSeconds / 60;
const wpm = minutes > 0 ? Math.round(words / minutes) : 0;
```

This is recalculated every animation frame so the live display is always accurate, not sampled.

---

### Canvas Chart

The WPM history chart is drawn from scratch on an HTML5 Canvas element every time a race finishes or the window resizes. It uses a `devicePixelRatio` scale to stay sharp on retina screens:

```js
canvas.width  = canvas.offsetWidth  * devicePixelRatio;
canvas.height = canvas.offsetHeight * devicePixelRatio;
ctx.scale(devicePixelRatio, devicePixelRatio);
```

The chart renders: a grid, a gradient-filled area, a smooth line connecting WPM samples, dot markers on each data point, and a label showing the most recent WPM — all with the Canvas 2D API and no charting library.

---

### Real-time Input Engine

Rather than comparing full strings on each keystroke, the engine scans from the start of the input to find the longest correct prefix — this correctly handles cases where fixing an earlier error ripples forward:

```js
let newPos = 0;
for (let i = 0; i < typed.length && i < target.length; i++) {
  if (typed[i] === target[i]) newPos = i + 1; else break;
}
```

Each character in the quote is a `<span>` with a `data-i` index attribute. Classes (`correct`, `wrong`, `pending`, `cursor`) are toggled per character every input event — giving instant per-character visual feedback without re-rendering the entire DOM.

---

## 📁 Project Structure

```
typeracer-x/
├── index.html      # Entire game — HTML + CSS + JS in one file
└── README.md
```

---

## 🔮 Future Improvements

- [ ] Multiplayer via WebSockets — real opponents in the same race
- [ ] Custom quote import — paste any text to race against
- [ ] Timed mode — type as many words as possible in 30 or 60 seconds
- [ ] Heatmap of most-mistyped characters
- [ ] Per-word WPM breakdown after each race
- [ ] Mobile soft-keyboard support

---

## 📄 License

MIT - free to use, modify, and learn from.

MIT — free to use, modify, and learn from.
