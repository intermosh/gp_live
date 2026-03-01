# GP_LIVE // SYS.CTRL

**Interactive Game Boy ROM visualizer with real-time audio reactivity and dual-window projection.**

GP_LIVE parses Game Boy (.gb/.gbc) ROMs to extract 8x8 tile graphics, then generates live audio-reactive visuals using the extracted sprite data. It features a controller interface for real-time manipulation (keyboard/gamepad) and a separate fullscreen projector window synchronized via `SharedArrayBuffer` for zero-latency communication.

## Key Features

- **ROM Parsing**: Extracts tile data from Game Boy ROMs with configurable scan ranges, bank selection, and quality thresholds.
- **Real-time Visual Engine**: 10 distinct visual patterns (dot matrix, scopes, ASCII fields, noise, etc.) with 9 post-processing effects (glitch, CRT, chromatic aberration, etc.).
- **Audio Reactivity**: Uses Web Audio API to analyze microphone input (16-band frequency data) and modulate visuals in real-time.
- **Dual-Window Architecture**: Controller UI manages state; projector window renders fullscreen visuals. Synchronized via `SharedArrayBuffer` (fallback to `postMessage`).
- **Gamepad Support**: Full mapping for Xbox/PlayStation-style controllers (analog sticks, D-pad, triggers, face buttons).
- **Sprite Sheet Caching**: Efficiently renders ROM tiles using offscreen canvas sprite sheets with palette-based colorization.
- **Cyberpunk Aesthetic**: Dark UI with neon accents, designed for live performance.

## Technical Stack

- **Frontend**: Vanilla JavaScript (ES6+), HTML5 Canvas, CSS3 (custom properties, grid).
- **Communication**: `SharedArrayBuffer` (SAB) for high-frequency state sync; `postMessage` fallback.
- **Audio**: Web Audio API (`AnalyserNode` for FFT, `getUserMedia` for microphone).
- **Graphics**: 2D Canvas API with `drawImage` sprite batching, `createImageBitmap` for preview frames.
- **ROM Processing**: Custom 2bpp tile decoder with quality scoring (coherence, color variety, fill ratio) and hash-based deduplication.

## Usage

1. Open `index.html` in a modern browser (Chrome/Edge recommended for SAB support).
2. Click **"1. Init Audio Engine"** and allow microphone access.
3. Drag & drop a `.gb` or `.gbc` ROM file into the **"GB ROM CARTRIDGE"** panel.
4. Adjust scan settings (mode, quality) and click **"RESCAN ROM"** if needed.
5. Click **"2. Open Projector"** to launch the fullscreen visual output.
6. Use keyboard (1-0, Q-P, A-L, Z-M, Space) or gamepad to switch palettes, patterns, effects, and speed.
7. Left stick Y-axis controls microphone gain; right stick X cycles patterns.

## Requirements

- Browser with `SharedArrayBuffer` support (Chrome/Edge 92+). Firefox requires `about:config` adjustments.
- HTTPS or `localhost` (SAB requires cross-origin isolation headers: `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp`).
- Gamepad (optional) for enhanced control.

## Architecture Overview

```
┌─────────────────┐     ┌─────────────────────┐     ┌─────────────────┐
│   Controller    │────▶│  SharedArrayBuffer  │◀────│   Projector     │
│   (UI + State)  │     │   (64-byte struct)  │     │   (Renderer)    │
└─────────────────┘     └─────────────────────┘     └─────────────────┘
        │                         │                         │
        │ postMessage (fallback)  │                         │
        └─────────────────────────┘                         │
                                                             │
        ┌─────────────────┐     ┌─────────────────────┐     │
        │   Audio Engine  │────▶│   FFT (16 bands)    │─────┘
        │   (Mic Input)   │     │                     │
        └─────────────────┘     └─────────────────────┘
```

**State Structure (SAB, 64 bytes)**:
```
Offset  Size  Field
0       1     sync counter
1       1     volume (0-255)
2-17    16    frequency bands (0-255)
18      1     palette (0-9)
19      1     pattern (0-9)
20      1     effect (0-8)
21      1     loop speed (0-6)
22      1     space trigger (incremented on burst)
```

## Performance

- **Controller**: Renders preview at 15 FPS (throttled), updates UI at 60 Hz.
- **Projector**: Renders at display refresh rate (60+ FPS), sends 10 FPS preview bitmap to controller.
- **ROM Parsing**: Scans up to 512 tiles; quality filter reduces CPU load. Caches sprite sheets per palette.

## Limitations

- SAB requires cross-origin isolation headers (may need server config for production).
- Maximum 512 unique tiles cached (excess discarded by quality score).
- No ROM emulation; only tile extraction for visual generation.

## License

copyleft © 2026 GP_LIVE Project. Built for live audiovisual performances.
