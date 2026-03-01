```markdown
# GP_LIVE // SYS.CTRL

> Real-time audio-reactive visual performance system with Game Boy ROM integration,
> dual-window projector sync, and gamepad control. Built for live AV performance.

---

## Overview

GP_LIVE is a single-file browser-based VJ tool designed for live audiovisual
performance. It captures microphone input, analyzes the frequency spectrum in
real time, and drives a set of generative visual patterns on a dedicated
projector window — all from a local HTML file with zero dependencies.

The controller UI and the projector output run as separate browser windows,
synchronized via `SharedArrayBuffer` (with `postMessage` fallback), achieving
sub-frame latency between input and visual output.

---

## Features

### Audio Engine
- Microphone input via Web Audio API (`getUserMedia`)
- 16-band FFT frequency analysis (`AnalyserNode`, `fftSize: 64`)
- Adjustable gain (0–300%) via slider or left analog stick
- VU meter display on controller UI

### Visual System
- **10 Patterns** — Dot Matrix, Circular Scope, ASCII Field, ANSI Grid,
  Perspective Floor, Sine Waveform, Binary Rain, Concentric Polygons,
  Target Lock, Noise Static
- **10 Palettes** — Cyberpunk, Matrix, Vaporwave, Meltdown, Mirror's Edge,
  Noir, Tron, Hazard, Deep Space, Panic
- **9 Post-processing Effects** — Chromatic Aberration, CRT Scanlines,
  Glitch Slice, Invert Block, Pixelate, Feedback Zoom, Hue Shift, Chaos
- **ASCII/ANSI Burst System** — 5 procedural burst types triggered on demand,
  driven by a deterministic hash function (no `Math.random` in hot path)

### Game Boy ROM Integration
- Parses `.gb` / `.gbc` cartridge headers (title, MBC type, bank count,
  header checksum validation)
- Extracts 2bpp tile graphics using a quality scoring algorithm
  (coherence, row variance, color variety, fill ratio)
- Deduplicates tiles via 32-bit hash; supports up to 512 unique tiles
- Scan modes: Auto (skip header), Bank Select, Manual Offset
- Sprite sheet architecture: tiles baked into `OffscreenCanvas` at load time,
  drawn via single `drawImage` GPU blit per tile at runtime
- Per-palette sheet caching (max 10 cached sheets, LRU eviction)
- ROM tiles replace generative glyphs across all patterns and burst types

### Dual-Window Sync
- Controller opens a dedicated Projector window via `window.open()`
- Sync via `SharedArrayBuffer` (requires COOP/COEP headers) with
  automatic `postMessage` fallback
- Projector sends back `ImageBitmap` thumbnail frames (320×180 @ ~10fps)
  to the controller preview panel via transferable objects (zero-copy)

### Input
- **Keyboard** — full mapping across all patterns, palettes, effects, loop speed
- **Gamepad** (Web Gamepad API)
  - LB/RB → Palette cycle
  - LT/RT → Pattern cycle
  - D-pad ◄► → Effect cycle
  - D-pad ▲▼ → Speed adjust
  - Left Stick Y → Mic gain
  - Right Stick X → Pattern (analog threshold snap)
  - A / B → ASCII Burst
  - Start → Reset to defaults

---

## Architecture

```
BackendController
├── AudioEngine          (Web Audio API, FFT analysis)
├── KeyboardHandler      (discrete key→state mapping)
├── GamepadHandler       (rAF-polled, edge-debounced buttons, analog axes)
├── SabTransport         (SharedArrayBuffer write / postMessage fallback)
├── PreviewPanel         (320×180 thumbnail, 15fps local or projector sync)
├── BackendUI            (DOM updates, VU meter, key indicators)
└── RomLoader
    ├── RomParser        (header decode, 2bpp tile extraction, quality score)
    └── RomAtlas         (sprite sheet builder, palette cache, draw API)

ProjectorController
├── SabTransport         (SharedArrayBuffer read)
├── Renderer             (pattern draw, burst system, effects)
└── RomAtlas             (reconstructed from serialized tile transfer)
```

---

## Keyboard Reference

| Group       | Keys     | Action              |
|-------------|----------|---------------------|
| Palettes    | `1` – `0`| Select color theme  |
| Patterns    | `Q` – `P`| Select visual mode  |
| Effects     | `A` – `L`| Apply post-process  |
| Speed       | `Z` – `M`| Adjust time scale   |
| Burst       | `Space`  | Trigger ASCII burst |

---

## Requirements

- Modern Chromium-based browser (Chrome 89+, Edge 89+)
- Microphone access
- For `SharedArrayBuffer`: server must send COOP/COEP headers,
  or use a local dev server with the following headers:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

> **No headers needed** if running locally via `file://` in Chrome with
> `--enable-features=SharedArrayBuffer` flag, or via a tool like
> [**servor**](https://github.com/lukejacksonn/servor) or **vite**.

---

## Usage

```bash
# Option A — static file server with correct headers (recommended)
npx servor . --cors

# Option B — Python
python3 -m http.server 8080

# Option C — direct file open (SharedArrayBuffer may fall back to postMessage)
open index.html
```

1. Open `index.html` in browser
2. Click **1. Init Audio Engine** — grant microphone permission
3. Click **2. Open Projector** — a second window opens as the output display
4. Connect a gamepad (optional)
5. Load a `.gb` / `.gbc` ROM via drag & drop (optional)
6. Perform

---

## ROM Tile Extraction — Technical Notes

Tiles are decoded from raw 2bpp Game Boy format:

```
For each 16-byte block (one 8×8 tile):
  For each row (2 bytes: lo, hi):
    For each bit 7→0:
      shade = (hi[bit] << 1) | lo[bit]   → 0..3
```

Quality scoring weights:

| Signal              | Weight |
|---------------------|--------|
| Horizontal coherence| 40%    |
| Row variance        | 25%    |
| Color variety       | 20%    |
| Fill ratio          | 15%    |

Tiles scoring below the quality threshold (default 35%) are discarded.
Duplicates are removed via hash before sorting by score descending.

---

## License

MIT
```
