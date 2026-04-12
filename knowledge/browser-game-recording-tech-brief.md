---
domain: browser-game-recording-tech
updated: 2026-04-12
confidence: medium-high
---

# Browser Game + Recording Technology Stack — Knowledge Brief

## Executive Summary

As of April 2026, Phaser has just released v4.0.0 "Caladan" (April 10, 2026), making v3.90.0 "Tsugumi" (May 2025) the final v3 release. The official Vue 3 templates exist for Phaser 3 (JS + TS variants via Vite), but v4 templates for Vue have not yet been confirmed. The Browser Gamepad API has ~83% cross-browser compatibility with Xbox controllers working best; DualSense and Switch Pro have quirks. MediaRecorder + canvas.captureStream() works for game recording but has significant gotchas around background tabs, WebM metadata, and real-time-only capture. ffmpeg.wasm (v0.12.15) can convert WebM to MP4 in-browser but is 5-24x slower than native and has memory ceilings. The Phavuer library (v0.19.7, April 2026) is the most active Vue 3 + Phaser integration wrapper.

---

## 1. Phaser.js Ecosystem

### Version History (Current)

| Version | Codename | Date | Status |
|---------|----------|------|--------|
| **4.0.0** | Caladan | April 10, 2026 | **Current stable** (just released) |
| 4.0.0-rc.6 | — | Dec 23, 2025 | Last release candidate |
| 4.0.0-rc.1 | — | Apr 2, 2025 | First release candidate |
| 4.0.0-beta.4 | — | Jan 2025 | Last beta |
| **3.90.0** | Tsugumi | May 23, 2025 | **Final v3 release** — no further v3 development |
| 3.88.x | — | Pre-May 2025 | Previous stable v3 |

### Phaser 4 Key Changes

- **New Render Node Architecture**: Replaced v3's pipeline system with a node-based renderer. Each render node handles a single task, WebGL state is fully managed, context restoration is built in.
- **Filter System**: FX and Masks from v3 merged into a single Filter system applicable to any game object or camera. Ships with Blur, Glow, Shadow, Pixelate, ColorMatrix, Bloom, Vignette, Wipe, ImageLight, GradientMap, Quantize, Blend.
- **SpriteGPULayer**: Can render 1 million sprites in a single draw call — up to 100x faster than standard rendering.
- **Breaking Changes**: Renderer, tint system, FX/Masks, Shader API, lighting all changed. Point, Mesh, BitmapMask classes removed.
- **Migration**: Official Migration Guide + dedicated AI migration skill file in the repo's `skills/` folder.

### Phaser 3 Final State (v3.90.0)

- Rounded corners for Rectangle Shape Game Objects (`setRounded()`)
- New angle utility methods (clockwise, counter-clockwise, shortest distance)
- Firefox fallback for positional audio (WebAudioSound follow)
- Drop-in replacement for v3.88 games
- **Explicitly stated as the last v3 release** — all future development is v4 only

### Community Health

- **GitHub Stars**: ~39,300
- **Ecosystem**: 2,000+ code examples, extensive API docs, active Discord + forums
- **Maintenance**: Commercially developed by Phaser Studio Inc + open source community
- **Decade of active development** with tens of thousands of games shipped
- **npm**: `phaser` package; latest tag is now v4.0.0

### Key Plugins

| Plugin | Purpose | Status |
|--------|---------|--------|
| **phaser3-rex-plugins** (rexrainbow) | 100+ plugins: UI, board systems, BBCode text, gestures, etc. | v1.80.19 (active, requires Phaser >= 3.80.0) |
| **phaser3-merged-input** | Unified keyboard + gamepad input mapping | v1.10.1 (Aug 2025, actively maintained) |
| **Built-in GamepadPlugin** | Native gamepad support (up to 4 pads) | Core Phaser, well-documented |
| **Built-in Animation Manager** | Frame-based sprite animation, yoyo, blending | Core Phaser |
| **Built-in Camera System** | Zoom, shake, flash, fade, follow, scroll | Core Phaser |
| **Camera3DPlugin** | 3D billboard sprites with z-axis perspective | Official plugin (v3 era) |

### Important Decision: v3 vs v4

For new projects starting April 2026: Phaser 4 is the official recommendation, but the ecosystem (plugins like rex, merged-input, community templates) is still largely v3-targeted. v4 keeps most of the v3 public API, so migration is feasible but non-trivial for renderer-dependent code. **If depending on rex plugins or community Vue templates, verify v4 compatibility first.**

---

## 2. Browser Gamepad API

### Specification Status

- **W3C Working Draft** (Web Applications Working Group, Recommendation track)
- Normalizes all controllers to "Standard Gamepad" layout (modeled after Xbox 360)
- Extensions spec adds vibrationActuator support

### Browser Support Matrix

| Browser | Version | Standard Mapping | Vibration | Notes |
|---------|---------|-----------------|-----------|-------|
| **Chrome** | 21+ (tested through 145) | Full | Yes | Best overall support |
| **Edge** | 12+ | Full | Yes | Chromium-based, matches Chrome |
| **Firefox** | 29+ (tested through 147) | Full | Partial/spotty | Some recent controller detection issues reported |
| **Safari** | 10.1+ (current: 26.x) | Full | Added in STP 162 | Known USB joystick issues in Safari 16/Ventura; WKWebView requires firstResponder |

**Overall compatibility score**: ~83/100 (LambdaTest)

### Controller-Specific Behavior

| Controller | USB/BT | Browser Mapping | Quirks |
|-----------|--------|----------------|--------|
| **Xbox (XInput)** | Both | Native standard mapping | Best supported; all buttons, analog triggers, sticks register correctly |
| **DualSense (PS5)** | USB-C / BT | Remapped to Xbox layout (Cross=A, Circle=B) | Adaptive haptics require Chrome/Edge only; touchpad registers as single click button, not touch surface; haptic feedback not available in Firefox/Safari |
| **Switch Pro** | USB / BT | Standard mapping | No system-wide deadzone adjustment on Switch itself; some button mapping inconsistencies across browsers |

### Deadzone Best Practices

**Recommended approach: Radial deadzone with rescaling**

```
// Radial deadzone with smooth rescaling
magnitude = sqrt(x*x + y*y)
if (magnitude < deadzone) {
  x = 0; y = 0;
} else {
  // Rescale to eliminate dead jump at threshold
  normalized = { x/magnitude, y/magnitude }
  scaledMag = (magnitude - deadzone) / (1 - deadzone)
  x = normalized.x * scaledMag
  y = normalized.y * scaledMag
}
```

**Practical deadzone values**:
- 0.00-0.04: Minimal (competitive players, new controllers)
- 0.05-0.12: Typical (masks light electrical noise)
- 0.13-0.25: Noticeable (hides larger offsets and worn sticks)

**Testing methodology**: Let controller sit untouched 30-60 seconds to collect idle data. If idle values drift in a circle pattern, use radial deadzone. If they form an ellipse, per-axis filtering may work better.

---

## 3. MediaRecorder API

### Browser Support

| Browser | Version | Container | Video Codecs | Audio Codecs |
|---------|---------|-----------|-------------|-------------|
| **Chrome** | 47+ (126+ for extended codecs) | WebM, MP4 | VP8, VP9, H.264, AV1 (126+), **HEVC (136+)** | Opus (48kHz) |
| **Firefox** | 30+ | WebM | VP8 only | Vorbis, Opus (44.1kHz) |
| **Safari** | 14.1+ (current: 26.x) | MP4 | H.264 | AAC, ALAC (STP 214+), PCM (STP 214+) |
| **Edge** | Chromium-based | Same as Chrome | Same as Chrome | Same as Chrome |

### Chrome 136 Update (2025-2026)

- Added HEVC encoding via `hvc1` codec string
- Added `hev1` and `avc3` codecs supporting variable-resolution video in MP4
- Most comprehensive MediaRecorder codec support of any browser

### canvas.captureStream() for Game Recording

**How it works**: `canvas.captureStream(fps)` creates a MediaStream from any canvas (2D, WebGL, WebGPU). MediaRecorder records this stream.

**Critical Limitations and Gotchas**:

1. **Real-time only**: MediaRecorder records wall-clock time. A 3-minute recording always takes 3 minutes. No frame-by-frame capture support in the spec (W3C issue #213 open).

2. **Background tab throttling**: Chrome reduces requestAnimationFrame to ~1fps in background tabs. Firefox similar. **If user switches tabs during recording, output will be choppy.** Workaround: Use getDisplayMedia() (OS-level capture) which is not affected by tab visibility.

3. **WebM duration metadata**: MediaRecorder-produced WebM files sometimes have incorrect/missing duration metadata, causing seek issues in players.

4. **Large canvas on Android**: Canvas captureStream() with resolutions above ~640x480 can cause MediaRecorder to stop abruptly on Android Chrome. Desktop Chrome handles 1280x720+ fine.

5. **Firefox memory leak**: Adding an audio track to a canvas stream via `addTrack()` causes constantly increasing RAM usage in Firefox — can exhaust system RAM in under a minute depending on canvas dimensions.

6. **Firefox bitmaprenderer**: Canvas using "bitmaprenderer" context with `transferFromImageBitmap` is not recorded by MediaRecorder in Firefox (Bug 1557980).

7. **Codec detection**: Always use `MediaRecorder.isTypeSupported(mimeType)` before recording. Cross-browser codec support varies dramatically.

### Recommended Strategy for Game Recording

- **Primary target**: Chrome/Edge with VP9 or H.264 in WebM/MP4 container
- **Fallback**: VP8 in WebM for Firefox
- **Safari**: H.264 in MP4 (limited but functional)
- **Always detect codec support** at runtime with `isTypeSupported()`
- **Warn users about tab switching** or use visibility API to pause recording
- **Post-process WebM** with ffmpeg.wasm for duration fix and MP4 conversion

---

## 4. Vue 3 + Phaser Integration Patterns

### Official Templates (Phaser Studio)

| Template | URL | Bundler | Language |
|----------|-----|---------|----------|
| **template-vue** | `phaserjs/template-vue` | Vite | JavaScript |
| **template-vue-ts** | `phaserjs/template-vue-ts` | Vite | TypeScript |

**Features**: Vue-to-Phaser communication bridge, hot-reloading, production build scripts.

**Status**: These are v3 templates. As of April 12, 2026 (2 days after Phaser 4 release), v4-specific Vue templates have not been announced yet. Given Phaser 4 keeps most of the v3 API, the v3 templates may work with minor modifications.

### Community Libraries

| Library | Version | Last Updated | Approach |
|---------|---------|-------------|----------|
| **Phavuer** (`laineus/phavuer`) | 0.19.7 | April 10, 2026 | Declarative wrapper — control Phaser objects via Vue components |
| **vue-phaser3** (`Sun0fABeach/vue-phaser3`) | — | Older | Webpack template with Pinia store bridging |

**Phavuer** is the most actively maintained integration library. It wraps Phaser 3 game objects as Vue 3 components, enabling declarative rendering of game scenes. Updated within days of this writing.

### Recommended Integration Pattern (2026)

1. **For new projects**: Use the official `phaserjs/template-vue-ts` as starting point with Vite
2. **Communication bridge**: EventEmitter pattern (Phaser emits events, Vue listens) or shared Pinia store
3. **Separation of concerns**: Vue handles UI overlays (menus, HUD, settings). Phaser handles the game canvas. They communicate via events, not direct DOM manipulation.
4. **If declarative game dev desired**: Phavuer wraps Phaser objects as Vue components, but adds abstraction layer — evaluate whether the convenience outweighs the coupling.

### Common Pitfalls

1. **Lifecycle mismatch**: Phaser's scene lifecycle != Vue component lifecycle. Destroying a Vue component doesn't automatically clean up Phaser scenes. Must manually call `scene.destroy()` or `game.destroy()`.
2. **Reactivity overhead**: Don't make Phaser game objects reactive (Vue proxies). It adds overhead and can cause unexpected behavior. Use `shallowRef` or `markRaw` for Phaser references stored in Vue state.
3. **Double rendering loops**: Vue's reactivity system and Phaser's game loop are independent. Avoid triggering Vue re-renders on every game frame.
4. **Canvas sizing**: Phaser manages its own canvas. Coordinate with Vue's layout system carefully, especially in responsive designs.

---

## 5. ffmpeg.wasm

### Current Version

| Package | Version | Last Published |
|---------|---------|---------------|
| `@ffmpeg/ffmpeg` | 0.12.15 | Jan 7, 2025 |
| `@ffmpeg/core` (single-thread) | 0.12.x | Paired with ffmpeg |
| `@ffmpeg/core-mt` (multi-thread) | 0.12.x | Paired with ffmpeg |

### Bundle Size

- **ffmpeg-core.wasm**: ~22 MB (full build)
- Can be reduced with custom builds (strip unused codecs/muxers)
- Loaded on-demand, not part of initial JS bundle

### Browser Requirements

| Requirement | Single-Thread | Multi-Thread |
|-------------|--------------|-------------|
| SharedArrayBuffer | **Not required** | **Required** |
| COOP header | Not required | `same-origin` |
| COEP header | Not required | `require-corp` |
| Web Workers | Recommended | Required |
| Chrome | All modern | 89+ |
| Firefox | All modern | 79+ |
| Safari | All modern | 15.2+ |

### WebM-to-MP4 Conversion

**Reliability**: Works but with caveats.

- **Remux (copy codecs)**: Fast, reliable if source codecs are MP4-compatible (H.264 video, AAC/Opus audio). Simply changes container.
- **Transcode (re-encode)**: Works but slow. VP8/VP9 WebM to H.264 MP4 requires full re-encode.
- **Performance**: 5-24x slower than native FFmpeg. A 30-second native transcode can take 5-10 minutes in WASM. H.264 1080p encoding: ~25fps on MacBook Pro 2018.
- **No GPU acceleration**: WebAssembly sandbox prevents hardware encoding access.
- **Memory ceilings**: Safari ~500MB before crash, Chrome ~1GB before slowdown, Firefox ~800MB. Large videos will fail.

### Known Issues

1. **Error handling is poor**: ffmpeg.wasm "almost never throws an error" — errors are hidden and only surface when the output file is played. Always validate output.
2. **No streaming input**: Must load entire file into memory before processing (as of 2025).
3. **Main thread blocking**: Even with Web Workers, WASM initialization can block the main thread.
4. **CORS headers for multi-thread**: Many hosting environments (GitHub Pages, some CDNs) don't set COOP/COEP headers by default. Single-thread works everywhere but is slower.

### Practical Recommendation

For game recording WebM-to-MP4 conversion:
- **Short clips (< 30s)**: ffmpeg.wasm single-thread is acceptable. Conversion takes seconds to low minutes.
- **Medium clips (30s-5min)**: Use multi-thread with proper headers. Conversion takes minutes.
- **Long recordings (5min+)**: Consider server-side conversion or offering WebM download directly. Browser memory limits become a real constraint.
- **Alternative**: If recording in Chrome 126+, record directly to MP4 container with H.264 codec via MediaRecorder, eliminating the need for ffmpeg.wasm conversion entirely.

---

## Cross-Cutting Recommendations

1. **Target Chrome/Edge first** — best MediaRecorder codec support, best Gamepad API support, best ffmpeg.wasm multi-thread support.
2. **Record directly to MP4/H.264 in Chrome 126+** to avoid WebM-to-MP4 conversion entirely.
3. **Phaser 4 is the future** but v3 ecosystem (rex plugins, Vue templates, Phavuer) is more mature. For a project starting now, evaluate whether v4 breaking changes are worth the performance gains.
4. **Radial deadzone with rescaling** is the industry standard for gamepad input.
5. **Tab visibility is a recording risk** — implement Page Visibility API detection and warn/pause when tab is backgrounded.

---

## Sources

Research conducted April 12, 2026 via web search across npm, GitHub, MDN, caniuse, Phaser.io, W3C specs, and developer blogs.

Key references:
- [Phaser GitHub Releases](https://github.com/phaserjs/phaser/releases)
- [Phaser v4.0.0 Download](https://phaser.io/download/release/v4.0.0)
- [Phaser v3.90 Release](https://phaser.io/news/2025/05/phaser-v390-released)
- [Official Vue 3 Template](https://github.com/phaserjs/template-vue)
- [Official Vue 3 TS Template](https://github.com/phaserjs/template-vue-ts)
- [Phavuer Library](https://github.com/laineus/phavuer)
- [phaser3-rex-plugins](https://github.com/rexrainbow/phaser3-rex-notes)
- [phaser3-merged-input](https://github.com/GaryStanton/phaser3-merged-input)
- [W3C Gamepad Spec](https://www.w3.org/TR/gamepad/)
- [Gamepad API - Can I Use](https://caniuse.com/gamepad)
- [HTML5 Gamepad API Developer Guide (2026)](https://gamepadtester.pro/the-html5-gamepad-api-a-developers-guide-to-browser-controllers/)
- [Thumbstick Deadzone Techniques](https://github.com/Minimuino/thumbstick-deadzones)
- [Gamepad Deadzone Guide (2026)](https://gamepadtest.app/guides/dead-zone-fix)
- [MDN Gamepad API](https://developer.mozilla.org/en-US/docs/Web/API/Gamepad_API)
- [MDN MediaRecorder](https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder)
- [MediaRecorder API - Can I Use](https://caniuse.com/mediarecorder)
- [Chrome 136 Beta](https://developer.chrome.com/blog/chrome-136-beta)
- [MediaRecorder Safari Support](https://webkit.org/blog/11353/mediarecorder-api/)
- [Canvas captureStream Gotchas](https://sendrec.eu/blog/why-canvas-breaks-your-screen-recorder/)
- [W3C MediaCapture Record Issue #213](https://github.com/w3c/mediacapture-record/issues/213)
- [ffmpeg.wasm GitHub](https://github.com/ffmpegwasm/ffmpeg.wasm)
- [@ffmpeg/ffmpeg npm](https://www.npmjs.com/package/@ffmpeg/ffmpeg)
- [ffmpeg.wasm Performance](https://ffmpegwasm.netlify.app/docs/performance/)
- [ffmpeg.wasm GPU Limitations](https://dayverse.id/en/articles/why-ffmpeg-wasm-fails-leverage-gpu-acceleration/)
