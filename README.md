# Harmony Music Player

A fully-featured, single-file music player web application built with HTML, CSS, and Vanilla JavaScript — no frameworks, no build step, no npm. The entire player — 3408 lines of markup, CSS variables, keyframe animations, and JavaScript logic — ships as one self-contained `index.html` file. It renders a two-column glassmorphism UI with a real-time playback engine, animated album art, a playlist manager with search and per-item actions, a lyrics panel, an equalizer visualizer, theme switching, and a persistent settings sidebar, all wired together through the Web Audio API and the browser's native `<audio>` element.

---

## What This Project Does

The page opens with a dark-themed glass-panel layout (`backdrop-filter: blur(20px)`) that slides up on load via a `slideUp` cubic-bezier entrance animation. The left column handles playback — album art with a 3D hover tilt and rotating animation when playing, a song info block, a scrubbing progress bar, and a full control row. The right column handles playlist management — a searchable scrollable song list, playlist CRUD actions, and a stats bar. A persistent notification badge on the favorites button tracks the liked song count. A sidebar panel (toggled via a header button) exposes equalizer controls, audio settings, and theme options. Everything updates in real time as songs play, are skipped, or are added and removed.

---

## UI Architecture — Two-Column Layout

The `.music-player` wrapper is a `max-width: 1200px` glassmorphism card centered on a dark tri-stop gradient body (`#0f0c29 → #302b63 → #24243e`). Inside, `.main-content` uses `display: grid; grid-template-columns: 1fr 1fr` — collapsing to single-column below 900px via a media query breakpoint. Every subsection (`song-info`, `progress-area`, `player-controls`, `playlist-management`, `current-playlist`) is an independently animated card — each with `background: rgba(255,255,255,0.05)`, `border-radius: 20px`, `border: 1px solid rgba(255,255,255,0.1)`, and its own `slideInLeft` / `slideInRight` / `fadeIn` entrance keyframe at a staggered `animation-delay`.

---

## CSS Variable System — 17 Design Tokens

All colors, gradients, shadows, and transitions are defined as CSS custom properties on `:root`, making every aspect of the theme overridable from one location:

```css
:root {
  --primary-gradient:   linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  --secondary-gradient: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  --dark-gradient:      linear-gradient(135deg, #0f0c29 0%, #302b63 50%, #24243e 100%);
  --light-gradient:     linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  --primary-color:      #667eea;
  --secondary-color:    #764ba2;
  --accent-color:       #f5576c;
  --success-color:      #4cd964;
  --warning-color:      #ffcc00;
  --danger-color:       #ff4757;
  --card-dark:          rgba(255, 255, 255, 0.08);
  --card-light:         rgba(255, 255, 255, 0.92);
  --shadow-dark:        0 20px 40px rgba(0, 0, 0, 0.5);
  --shadow-light:       0 15px 30px rgba(0, 0, 0, 0.1);
  --transition-slow:    0.5s cubic-bezier(0.68, -0.55, 0.265, 1.55);
  --transition-medium:  0.3s ease;
  --transition-fast:    0.2s ease;
}
```

The light theme is applied by toggling `body.light-theme`, which overrides `background`, `color`, and card backgrounds across every component without touching any other rule.

---

## Album Art — Vinyl + 3D Tilt + Rotation

The `.album-art-container` uses CSS `perspective: 1000px` to enable GPU-accelerated 3D transforms. On hover, the `.album-art` applies `rotateY(10deg) rotateX(5deg) scale(1.03)` — giving the cover art a physical, card-like tilt. When a track is playing, the `<img>` inside receives the class `.playing`, which activates:

```css
@keyframes rotateAlbumArt {
  0%   { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
```

This makes the album art spin continuously like a vinyl record at 20 seconds per rotation. A `.vinyl-overlay` radial gradient darkens the outer edges, and two nested divs (`.vinyl-center`, `.vinyl-hole`) create a centered vinyl label + spindle hole in front of the spinning image.

---

## Progress Bar — Scrubbing + Shimmer + Thumb

The `.progress-bar` is a 10px tall rounded track. A `::before` pseudo-element runs a `shimmer` animation across it — shifting a 200%-width gradient from left to right at 3s/cycle — to give the unplayed portion a living, glowing texture. The `.progress` fill div grows via `width: X%` updated by JavaScript on `timeupdate` events. A `::after` pseudo-element on `.progress` renders as the draggable white thumb — 25×25px circle that scales up and pulses blue on hover.

---

## Control Buttons — Ripple + Pulse Ring

Each `.control-btn` (header) and `.player-btn` (playback controls) uses a `::before` circle that expands from `width: 0; height: 0` to `100%; 100%` on hover — creating a contained ripple effect without JavaScript. The central `.play-pause` button has an additional `::before` ring that runs:

```css
@keyframes pulseRing {
  0%   { transform: scale(0.8); opacity: 0.3; }
  70%  { transform: scale(1.1); opacity: 0.1; }
  100% { transform: scale(1.2); opacity: 0; }
}
```

This creates a continuous breathing glow ring behind the play button while music is running.

---

## Playlist — Search, CRUD, Per-Item Actions

The `.playlist-search` input filters the song list in real time via JavaScript `input` event — comparing `toLowerCase()` against each `.playlist-song-title` text and toggling `display: none` on non-matching `.playlist-item` rows. Each row has a `.playlist-item-actions` group (opacity 0, transitioning to 1 on row hover) containing per-song buttons: favorite/like, add to queue, and delete. Deleting a song removes its element from the DOM and triggers a `GSAP` exit animation (scale-to-zero, opacity-to-zero). Adding songs opens a styled modal form where the user inputs title, artist, album, genre, and an image URL — the new entry is constructed as a DOM node and appended to `.playlist-container` with a `fadeInUp` entrance animation.

---

## Floating Music Notes — Background Animation

A fixed `.music-notes` container (z-index: -1, pointer-events: none) holds multiple `.note` elements — actual music note emoji characters (`♪`, `♫`, `🎵`) absolutely positioned at randomized horizontal offsets. Each runs:

```css
@keyframes floatNote {
  0%   { transform: translateY(100vh) rotate(0deg); opacity: 0; }
  10%  { opacity: 0.3; }
  90%  { opacity: 0.3; }
  100% { transform: translateY(-100px) rotate(360deg); opacity: 0; }
}
```

Notes rise from below the viewport, fade in at 10%, hold opacity through 90% of the journey, then fade out as they exit the top — looping infinitely at durations staggered between 15s and 25s per note.

---

## Logo Shine Effect

The `.logo` box has a `::after` pseudo-element — a rotated `linear-gradient(to bottom right, rgba(255,255,255,0.3), transparent)` — that animates with `translateX` and `translateY` from `-100%` to `+100%` over 3 seconds. This creates a diagonal shine sweep across the logo that repeats every 3 seconds.

---

## Tech Stack

| Technology | Role |
|---|---|
| HTML5 | Single 3408-line self-contained file — all markup, inline `<style>`, and inline `<script>` |
| CSS3 | 17-token CSS variable system, glassmorphism cards, 10+ named `@keyframes`, light/dark theme toggle via `body.light-theme` class |
| JavaScript (Vanilla) | Web Audio API playback engine, `timeupdate` progress sync, playlist CRUD, DOM-based search filter, modal system, LocalStorage for favorites and settings persistence |
| Font Awesome 6.4.0 (CDN) | All icons — play, pause, skip, shuffle, repeat, heart, settings, equalizer, playlist controls |

---

## Project Structure

```
Music-Player/
├── index.html     # Complete 3408-line single-file application — CSS variable system (17 tokens), glassmorphism layout (backdrop-filter blur), two-column grid (collapses at 900px), album art with vinyl overlay + 3D tilt + rotation keyframe, progress bar with shimmer + draggable thumb pseudo-element, play-pause pulse ring, playlist with real-time search filter + per-item CRUD actions, floating music note background animation, logo shine sweep, light/dark theme toggle, sidebar panel for equalizer + settings, Web Audio API playback engine
└── README.md
```

---

## How to Run

1. Clone the repository
   ```bash
   git clone https://github.com/tripathipawan/Music-Player.git
   ```
2. Open `index.html` directly in any modern browser. No server, no npm, no build step — the entire application runs from a single file.

---

## Repository

[https://github.com/tripathipawan/Music-Player](https://github.com/tripathipawan/Music-Player)
