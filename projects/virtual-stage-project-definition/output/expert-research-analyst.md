# Expert Analysis: Research Analyst -- Virtual Stage

**Agent**: Research Analyst
**Project**: Virtual Stage
**Date**: 2026-04-12
**Confidence**: Medium-High (primary sources verified within 48 hours; market sizing from multiple corroborating reports)

---

## Research Summary

Virtual Stage occupies a genuinely unoccupied niche at the intersection of four established markets: pixel art tooling, gamepad-driven character control, browser-based creative tools, and short-form video content creation. No existing product combines gamepad-controlled pixel art characters on a 2D stage with integrated video recording. The closest tool, veadotube mini, handles expression switching only -- not movement, actions, or scene composition. The technology stack is viable: Phaser.js has strong gamepad support and a decade of production use, Chrome 126+ can record directly to MP4/H.264 without ffmpeg.wasm, and the LPC sprite sheet system provides a near-standard 64x64 4-direction format with extensive CC0/CC-BY-SA assets. The critical decision point is Phaser v3 vs. v4: v4.0.0 "Caladan" released April 10, 2026 (two days ago), offering dramatic rendering improvements but an ecosystem (Vue templates, rex plugins, merged-input) still anchored to v3. The VTuber/virtual avatar market ($5.38B growing to $7.26B) and the pixel art renaissance on social platforms create strong tailwinds, while upcoming platform restrictions on AI-generated content create a potential authenticity premium for hand-crafted pixel art performances.

---

## Domain Context

### Industry

Virtual Stage sits at the convergence of three distinct industries:

1. **Creative Tools for Content Creators**: The creator economy reached $37B in U.S. ad spend in 2025 (+26% YoY). Short-form video platforms (TikTok, YouTube Shorts, Instagram Reels) continue to drive demand for lightweight content production tools. The trend is toward browser-based, zero-install tools that lower the barrier to content creation.

2. **VTuber / Virtual Avatar Market**: Valued at $5.38B in 2025 with projections to $7.26B by end of 2026, this market has seen 189% growth in the VTuber category. The PNGTuber subcategory is growing particularly fast as a lower-barrier entry point -- creators use static or semi-animated 2D images as stream overlays rather than rigged 3D or Live2D models. Virtual Stage extends this concept from static overlays to full scene-based performance.

3. **Pixel Art / Retro Aesthetic Revival**: Pixel art games dominate indie gaming in 2025-2026, with the aesthetic experiencing what industry observers call a "renaissance." Active communities on TikTok (#pixelartloop), Instagram, and YouTube sustain demand for pixel art content. Notably, TikTok is expected to implement aggressive AI content restrictions by mid-2026, creating an authenticity premium for hand-crafted content formats like pixel art animation.

**Source**: pixel-art-machinima-puppeteering-brief.md, Sections 4 and 6.

### Key Trends

- **PNGTuber growth**: The simplest form of virtual avatar (switching between 2D images based on mic input) has exploded in adoption. veadotube mini is the de facto standard. This validates demand for lightweight, non-3D avatar tools.
- **Browser-first creative tools**: No dedicated browser-based machinima tool has emerged, despite browser capabilities now supporting game-quality rendering (Phaser, PixiJS), real-time audio capture (Web Audio API), and video recording (MediaRecorder). The gap is tooling, not technology.
- **Gamepad as creative input**: iClone Motion Director proves the gamepad-to-character-control-to-recording pipeline works in 3D. No equivalent exists in 2D. The Nyarupad VTS plugin for VTube Studio adds gamepad expression switching, confirming creator interest in gamepad-based avatar control.
- **Short-form video dominance**: MP4 is the preferred output format for TikTok/Reels/Shorts (better compression and platform support than GIF). Walk/idle animations at 8-12 FPS, run at 12-15 FPS, attack at 15-20 FPS are standard pixel art frame rates.
- **Authenticity premium**: As AI-generated content faces platform restrictions and audience skepticism, hand-controlled pixel art performances -- where a human physically puppeteers a character with a gamepad and speaks dialogue live -- carry an intrinsic authenticity signal.

**Source**: pixel-art-machinima-puppeteering-brief.md, Sections 2, 4, and 6.

### Regulations / Standards

- **LPC Sprite Sheet Standard**: The Universal LPC Spritesheet system is the closest thing to a standardized sprite format -- 64x64 pixel frames, 4-direction grid layout, dual-licensed under GNU GPL 3.0 and CC-BY-SA 3.0 (some assets CC0). A browser-based character generator exists at liberatedpixelcup.github.io. Animations include spellcast, slash, thrust, walk, shoot, hurt, bow, climb, run, and jump.
- **Sprite Sheet Conventions**: No formal standard, but strong industry conventions exist: 16x16, 32x32, or 64x64 frame sizes; PNG with alpha transparency; JSON metadata (Aseprite format widely adopted); power-of-two texture sizes (256, 512, 1024); 1-2px padding between frames.
- **Microphone/Camera Permissions**: Browser APIs require explicit user consent via getUserMedia(). Privacy implications for microphone access must be communicated clearly.
- **Content Licensing**: Any bundled assets must respect GPL/CC-BY-SA/CC0 licensing terms. Attribution requirements for LPC assets must be surfaced to users who use bundled characters.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 5; browser-game-recording-tech-brief.md, Section 3.

---

## Competitive Landscape

### veadotube mini
- **What it does**: The most widely used PNGTuber application. Switches between expression images (PNG, animated GIF, animated PNG, WebP) based on microphone input (talking/silent). Free (pay-what-you-want on itch.io). Cross-platform. Has a "pixelated" rendering toggle specifically for pixel art. Supports gamepad, keyboard, mouse, MIDI controller, and Stream Deck for expression switching. Unlimited expression slots. Integrates with OBS.
- **Strengths**: Extremely lightweight and accessible. Zero learning curve. Already adopted by the pixel art VTuber community. Active development -- a full version with scene editor, collaboration, scripting, and dynamic sprites is planned for 2026 release.
- **Weaknesses**: Expression switching only. Characters do not move, walk, interact, or perform actions. No stage/scene concept. No built-in recording (requires OBS). No spatial awareness -- characters cannot coexist in a shared scene.
- **Relevance**: Validates the market for lightweight pixel art avatar tools with gamepad input. Virtual Stage extends the concept from "expression overlay" to "full performance on a stage." The upcoming veadotube full version with its scene editor could become a future competitor if it adds gamepad-driven character movement, but its current trajectory focuses on dynamic sprites and node-based animation rather than gamepad puppeteering.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 2 (VTuber tools).

### Cartoon Animator 5.3 (Reallusion)
- **What it does**: Professional 2D animation software with a "Motion Pilot" feature for real-time puppet control and a "Puppet Stage" for scene-based recording. Supports importing 2D artwork and rigging it for animation.
- **Strengths**: Mature puppet pipeline with Motion Pilot for real-time recording. Puppet Stage provides scene composition. Professional output quality. Established user base.
- **Weaknesses**: Mouse-driven only -- no native gamepad support found. Not pixel-art-native (designed for vector/HD artwork; pixel art is possible but not the intended aesthetic). $149+ perpetual license. Desktop application, not browser-based. Significant learning curve for rigging.
- **Relevance**: Proves the "real-time puppet control recorded as animation" concept has market demand. Virtual Stage differentiates by being browser-based, gamepad-driven, pixel-art-native, and free.

**Source**: pixel-art-machinima-puppeteering-brief.md, Sections 1 and 6.

### iClone 8 + Motion Director (Reallusion)
- **What it does**: Professional 3D animation tool. Motion Director specifically supports gamepad and keyboard input to control 3D characters in real-time, recording the movement as animation data. iClone Video Mocap (2025) adds AI-based motion capture from ordinary video at ~$2.50 per motion.
- **Strengths**: The only existing tool that implements gamepad-to-character-to-recording in a stage environment. Professional 3D output. Mature pipeline.
- **Weaknesses**: 3D only -- no pixel art support. $199+/year (iClone 365) or $599 perpetual. Desktop application. Steep learning curve. Targets professional animators, not content creators.
- **Relevance**: Directly validates the gamepad puppeteering concept. Virtual Stage is essentially "iClone Motion Director for pixel art, in a browser, for free." The existence of Motion Director proves the input paradigm works; Virtual Stage applies it to a radically different aesthetic and audience.

**Source**: pixel-art-machinima-puppeteering-brief.md, Sections 1 and 6.

### Inochi2D + Puppetstring
- **What it does**: Open-source 2D puppet animation framework. Considered the best option for pixel art VTuber models specifically. Puppetstring is a companion app implementing webcam, mouse, and keyboard tracking for Inochi2D models. A Steam community guide exists specifically for pixel artists ("Pixel-artists' Handbook").
- **Strengths**: Open-source. Specifically acknowledged as viable for pixel art. Active community. Purpose-built for 2D puppet rigging and animation.
- **Weaknesses**: No gamepad support. No stage/scene concept. No built-in recording (requires OBS). Keyboard and mouse tracking only via Puppetstring. Early-stage compared to commercial alternatives. Requires rigging knowledge.
- **Relevance**: The most technically aligned open-source project. If Virtual Stage ever needs skeletal animation (beyond sprite sheet frame switching), Inochi2D's approach could inform the architecture. However, Inochi2D solves a different problem (puppet rigging) rather than the performance/recording problem Virtual Stage targets.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 2.

### Source Filmmaker (SFM) / s&box
- **What it does**: SFM is Valve's machinima tool built on the Source engine. s&box, the spiritual successor to Garry's Mod by Facepunch Studios, is launching April 28, 2026 on Steam, built on Source 2 with a Valve licensing deal for royalty-free game sales.
- **Strengths**: SFM has a massive existing machinima community (TF2 content dominates). s&box brings modern tooling and Source 2 visuals with an open modding framework.
- **Weaknesses**: SFM is effectively in maintenance mode with no significant updates. Both are 3D-only. Desktop applications. Steep learning curves. No pixel art support. s&box targets game creation and modding rather than content recording specifically.
- **Relevance**: Represents the "old guard" of machinima -- complex 3D tools with high barriers to entry. Virtual Stage is positioned as the opposite: simple, 2D, browser-based, pick-up-and-play. s&box launching two weeks from now could absorb some machinima creator attention, but it serves a fundamentally different audience and aesthetic.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 1.

### PNGTuber Plus / PNGTuber Maker
- **What it does**: Free/open-source PNGTuber tools. PNGTuber Plus adds bouncing, jiggling, and animated sprite sheets to the PNGTuber formula. PNGTuber Maker provides built-in backgrounds, accessories, and virtual camera output.
- **Strengths**: Free. Accessible. Growing community. PNGTuber Plus supports animated sprite sheets.
- **Weaknesses**: Expression-switching and idle animation only. No character movement. No stage/scene. No gamepad-driven control of character actions. Require OBS for recording.
- **Relevance**: Part of the same ecosystem as veadotube mini. Further validates demand for lightweight 2D avatar tools. Virtual Stage leapfrogs these by adding spatial movement, actions, multi-character stages, and integrated recording.

**Source**: pixel-art-machinima-puppeteering-brief.md, Sections 2 and 6.

### Adobe Character Animator
- **What it does**: Professional puppet animation tool (part of Creative Cloud). Uses webcam and microphone to drive puppet animation with face tracking, lip sync, and physics behaviors.
- **Strengths**: Professional quality. Real-time performance capture. Integrated with Adobe ecosystem.
- **Weaknesses**: Subscription-locked (Creative Cloud). Not pixel-art-native (users have created pixel art puppets, but it is not a promoted use case). No native gamepad support (partial MIDI controller support). Complex. Desktop-only.
- **Relevance**: Demonstrates that real-time puppet animation has professional market demand. Virtual Stage targets a completely different user (indie creator vs. professional animator) with a completely different tool philosophy (free browser tool vs. subscription desktop suite).

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 2.

### Pixel Art Puppet Tool (2b1r)
- **What it does**: After Effects-style puppet pin warping for pixel sprites (up to 128x128, typically 64x64). Creates morphing/warping animations exported as PNG sprite sheets and GIFs. Reached v2.0 on itch.io.
- **Strengths**: Purpose-built for pixel art. Supports the standard 64x64 frame size. Produces sprite sheet output.
- **Weaknesses**: Offline tool for pre-baked animation production. No real-time control. No gamepad support. No stage or scene concept. Pin-warping approach can distort pixel art aesthetics.
- **Relevance**: Solves a different part of the pipeline (creating animated sprite sheets from static pixel art). Could be a complementary tool -- artists use Pixel Art Puppet Tool or Aseprite to create sprite sheets, then load them into Virtual Stage for performance.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 3.

### Competitive Gap Summary

| Capability | veadotube mini | Cartoon Animator | iClone Motion Dir. | Inochi2D | Virtual Stage |
|---|---|---|---|---|---|
| Pixel Art Native | Yes (toggle) | No | No (3D) | Possible | **Yes** |
| Gamepad Control | Expression only | No (mouse) | **Yes** | No | **Yes (full movement)** |
| Character Movement | No | Yes (mouse) | Yes | Partial | **Yes** |
| Multi-Character | No | Yes | Yes | No | **Yes (up to 4)** |
| Stage / Scene | No | Yes | Yes | No | **Yes** |
| Built-in Recording | No (OBS) | Yes | Yes | No (OBS) | **Yes** |
| Browser-Based | No | No | No | No | **Yes** |
| Free | Yes | No ($149+) | No ($199+/yr) | Yes | **Yes** |
| Voice Acting Integration | Mic trigger | Face track | Face track | Webcam | **Mic capture** |

**No existing tool fills all columns.** Virtual Stage is the only proposed solution that combines pixel art, gamepad control with full movement, multi-character staging, built-in recording, and browser-based free access.

---

## Technology Landscape

### Phaser.js: v3 vs. v4 Decision

This is the single most consequential technical decision for the project, and the timing is particularly challenging: Phaser 4.0.0 "Caladan" released on April 10, 2026 -- literally two days ago.

**Phaser 4 Advantages**:
- **SpriteGPULayer**: Can render 1 million sprites in a single draw call, up to 100x faster than standard rendering. This is overkill for Virtual Stage's needs (a handful of characters and props) but provides massive headroom.
- **New Filter System**: Merged FX and Masks into a unified Filter system with built-in Blur, Glow, Shadow, Pixelate, ColorMatrix, Bloom, Vignette, Wipe -- many of which would be valuable for scene transitions and visual effects.
- **Render Node Architecture**: Fully managed WebGL state with built-in context restoration. More robust for long recording sessions.
- **Future-proofing**: v3.90.0 is explicitly the final v3 release. All future development is v4 only. Starting on v3 means building on an EOL framework.

**Phaser 4 Risks**:
- **Ecosystem lag**: The official Vue 3 templates (`phaserjs/template-vue-ts`) target Phaser 3. As of April 12, 2026, v4-specific Vue templates have not been announced. Phaser 4 keeps most of the v3 public API, so the v3 templates "may work with minor modifications," but this is unverified.
- **Plugin compatibility**: phaser3-rex-plugins (v1.80.19, 100+ plugins including critical UI components) requires Phaser >= 3.80.0 and has no confirmed v4 support. phaser3-merged-input (v1.10.1, the best unified gamepad/keyboard input plugin) is explicitly v3-targeted.
- **Breaking changes**: Renderer, tint system, FX/Masks, Shader API, and lighting all changed. Point, Mesh, BitmapMask classes removed. Migration is feasible (official Migration Guide exists) but non-trivial for renderer-dependent code.
- **Phavuer compatibility**: Phavuer (v0.19.7, updated April 10, 2026 -- same day as v4 release) wraps Phaser 3 game objects as Vue components. Its v4 compatibility is unknown.
- **Community resources**: The vast majority of Phaser tutorials, examples, and Stack Overflow answers target v3. v4-specific learning resources are two days old.

**Assessment**: The intake document specifies Phaser v3. Given the ecosystem reality on April 12, 2026, this is the pragmatic choice. Starting with v3.90.0 (the final, most stable v3) allows use of the mature template, plugin, and community ecosystem. The Vue-Phaser integration pattern (mitt EventBridge, not Phavuer) minimizes Phaser-version coupling. Migration to v4 can happen once the ecosystem catches up (estimated 3-6 months), and since Virtual Stage does not depend on renderer-level customization, the migration surface should be manageable.

**Recommendation**: Start with Phaser 3.90.0. Use the official `phaserjs/template-vue-ts` as the project scaffold. Avoid deep dependencies on v3-specific renderer internals. Plan a v4 migration for Phase 5 or 6 (post-MVP) once rex plugins and Vue templates confirm v4 support.

**Source**: browser-game-recording-tech-brief.md, Section 1.

### MediaRecorder API: Direct MP4 Recording

A significant finding that simplifies the recording pipeline:

- **Chrome 126+** (current Chrome is well beyond 126) supports recording directly to MP4 container with H.264 video and Opus audio via MediaRecorder. This eliminates the need for ffmpeg.wasm conversion for the primary target browser.
- **Chrome 136+** added HEVC encoding (`hvc1` codec string) and variable-resolution MP4 support (`hev1`, `avc3`).
- **Firefox** only supports VP8 in WebM container (no MP4, no H.264).
- **Safari** supports H.264 in MP4 (limited but functional).

**Practical impact**: The intake document specified WebM output via MediaRecorder with potential ffmpeg.wasm conversion to MP4. The updated recommendation is:

1. **Primary path (Chrome/Edge)**: Record directly to MP4/H.264 via `canvas.captureStream()` + MediaRecorder with `video/mp4; codecs="avc1.42E01E"` (or similar). No post-processing needed. Output is immediately shareable on TikTok/YouTube/Instagram.
2. **Fallback path (Firefox)**: Record to WebM/VP8. Offer WebM download or use ffmpeg.wasm for MP4 conversion (acceptable for clips under 5 minutes).
3. **Safari**: Not targeted (per intake), but H.264/MP4 recording would work if support is ever added.

**Critical gotchas** that must be designed around:
- **Background tab throttling**: Chrome reduces requestAnimationFrame to ~1fps when the tab is backgrounded. If a user switches tabs during recording, the output will be choppy. Must implement Page Visibility API detection to warn or auto-pause.
- **WebM duration metadata**: MediaRecorder-produced WebM files sometimes have incorrect/missing duration, causing seek issues. Direct MP4 recording avoids this entirely.
- **Real-time only**: MediaRecorder records wall-clock time. A 3-minute scene takes 3 minutes to record. No "offline rendering" capability exists in the browser spec (W3C issue #213 remains open).
- **Codec detection**: Always use `MediaRecorder.isTypeSupported(mimeType)` at runtime. Never assume codec availability.

**Source**: browser-game-recording-tech-brief.md, Section 3.

### ffmpeg.wasm: Reduced Role

Given direct MP4 recording in Chrome 126+, ffmpeg.wasm's role shrinks from "required conversion layer" to "Firefox fallback and optional post-processing":

- **Version**: @ffmpeg/ffmpeg v0.12.15 (Jan 2025). 22 MB WASM core.
- **Performance**: 5-24x slower than native FFmpeg. A 30-second native transcode can take 5-10 minutes in WASM.
- **Memory ceilings**: Safari ~500MB, Chrome ~1GB, Firefox ~800MB before failure.
- **Recommendation**: Load ffmpeg.wasm on-demand only for Firefox users or users who want format conversion. Do not include it in the initial bundle. For clips over 5 minutes, offer WebM download directly rather than attempting in-browser conversion.

**Source**: browser-game-recording-tech-brief.md, Section 5.

### Browser Gamepad API

- **Specification**: W3C Working Draft, Recommendation track. Normalizes controllers to "Standard Gamepad" layout (Xbox 360 model).
- **Compatibility**: ~83% cross-browser. Chrome/Edge have the best and most consistent support.
- **Controller support**: Xbox controllers work best (native standard mapping). DualSense (PS5) is remapped to Xbox layout (Cross=A, Circle=B); adaptive haptics Chrome/Edge only; touchpad registers as single click. Switch Pro has some mapping inconsistencies across browsers.
- **Phaser integration**: Phaser's built-in GamepadPlugin supports up to 4 simultaneous gamepads. The phaser3-merged-input plugin (v1.10.1, actively maintained) provides unified keyboard + gamepad mapping.
- **Deadzone**: Industry standard is radial deadzone with rescaling. Recommended default: 0.05-0.12 (masks electrical noise without feeling sluggish). Should be user-configurable.

**Source**: browser-game-recording-tech-brief.md, Section 2.

### Vue 3 + Phaser Integration

- **Official templates**: `phaserjs/template-vue-ts` (Vite + TypeScript) targets Phaser 3. Provides Vue-to-Phaser communication bridge with hot-reloading.
- **Phavuer** (v0.19.7): Most active Vue 3 + Phaser wrapper. Declarative approach (Phaser objects as Vue components). Updated April 10, 2026, but wraps v3 objects. Adds an abstraction layer that may complicate the EventBridge pattern specified in the intake.
- **Recommended pattern (per intake)**: mitt EventEmitter bridge. Vue handles UI (menus, HUD, settings, recording controls). Phaser handles the game canvas. Communication via events only. This is the correct architecture -- it decouples Vue and Phaser lifecycles and makes future Phaser version migration easier.
- **Critical pitfalls**: Do not make Phaser game objects reactive (use `shallowRef` or `markRaw`). Do not trigger Vue re-renders on every game frame. Manually handle Phaser scene destruction when Vue components unmount.

**Source**: browser-game-recording-tech-brief.md, Section 4.

### Pixel Art Asset Ecosystem

- **LPC Spritesheet System**: 64x64 frames, 4 directions, extensive animation sets (walk, run, slash, thrust, spellcast, hurt, etc.). Browser-based character generator. Dual GPL 3.0 / CC-BY-SA 3.0 with some CC0 assets. This is the recommended starting format for Virtual Stage characters.
- **itch.io CC0 assets**: Dedicated category (itch.io/game-assets/free/tag-cc0/tag-pixel-art). Notable packs include 8-directional knight (30+ animations), animated pixel adventurer (rvros), and Sidescroller Fantasy (600+ sprites).
- **Aseprite export format**: PNG sprite sheet + JSON metadata is the de facto standard. Pixelorama also exports in this format. Virtual Stage should support Aseprite JSON import natively.
- **Frame size compatibility**: LPC uses 64x64, many itch.io packs use 32x32 or 16x16. Virtual Stage should support configurable frame sizes with 64x64 as default (matching LPC).

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 5.

---

## Key Findings

### 1. The niche is genuinely unoccupied -- this is not a "slightly better" play, it is a "first mover" play

After exhaustive analysis of machinima tools, VTuber/puppet tools, pixel art tools, and browser-based creative tools, no product exists that combines gamepad-controlled pixel art characters, a virtual stage environment, and integrated video recording. The gap analysis table in the knowledge brief confirms this across eight competing or adjacent tools. This is not a case of building a marginally better version of an existing product; it is creating a new product category.

**Confidence**: High. Two independent research sweeps (machinima tools and VTuber tools) converge on the same conclusion.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 6 ("Direct Competitors: None found").

### 2. veadotube's trajectory validates demand but leaves the performance gap open

veadotube mini's dominance in the PNGTuber space, combined with its gamepad support for expression switching, demonstrates that pixel art creators want gamepad-driven avatar control. The upcoming veadotube full version will add a scene editor and dynamic sprites, but its announced feature set focuses on node-based animation rather than gamepad-driven spatial movement. Virtual Stage's core differentiation -- characters that walk, run, emote, and interact on a stage, controlled by joysticks in real-time -- is not on veadotube's published roadmap.

**Confidence**: Medium-High. The full version's feature set could shift; monitor veadotube development updates.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 2.

### 3. Chrome 126+ direct MP4 recording eliminates the biggest technical risk

The intake document identified video recording as the core technical challenge and specified WebM output with potential ffmpeg.wasm conversion. The discovery that Chrome 126+ supports direct MP4/H.264 recording via MediaRecorder fundamentally de-risks the recording pipeline for the primary target browser. MP4 is the preferred format for TikTok, YouTube Shorts, and Instagram Reels. This means the MVP recording pipeline is: `canvas.captureStream()` -> `MediaRecorder (video/mp4; codecs=avc1)` -> downloadable MP4. No transcoding, no WASM, no post-processing.

**Confidence**: High. Verified via MDN documentation and Chrome release notes through v136.

**Source**: browser-game-recording-tech-brief.md, Section 3.

### 4. Phaser v3 is the correct choice today, but v4 migration should be planned

Phaser 4.0.0 released two days ago. The v4 improvements (SpriteGPULayer, Filter System, Render Node Architecture) are compelling but the ecosystem has not caught up: Vue templates, rex plugins, merged-input plugin, Phavuer, and community resources all target v3. Starting on v3.90.0 (the final, most polished v3 release) is the pragmatic choice. The EventBridge architecture (mitt, not Phavuer) minimizes Phaser-version coupling, making future migration tractable. The v4 Filter system (Pixelate, Glow, Bloom, Vignette) would be particularly valuable for scene effects, so migration should be planned once the ecosystem stabilizes.

**Confidence**: High. Based on direct analysis of Phaser release timeline and ecosystem compatibility.

**Source**: browser-game-recording-tech-brief.md, Section 1.

### 5. The LPC sprite sheet system provides a ready-made asset foundation

The Universal LPC Spritesheet system is effectively a standard for multi-directional pixel art character animation: 64x64 frames, 4-direction grid layout, extensive animation sets (walk, run, slash, thrust, spellcast, hurt, jump, climb). A browser-based character generator exists. The licensing (GPL 3.0 / CC-BY-SA 3.0, some CC0) is compatible with a free tool. Combined with the extensive CC0 pixel art on itch.io, Virtual Stage can launch with a meaningful asset library without commissioning custom art.

**Confidence**: High. LPC system verified via the character generator and itch.io asset listings.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 5.

### 6. The "authentic content" premium is a tailwind that could accelerate adoption

TikTok is expected to implement aggressive AI content restrictions by mid-2026. Pixel art content produced through live gamepad performance with real voice acting is inherently "authentic human content" -- it cannot be easily replicated by AI generation. This positions Virtual Stage content as naturally high-signal in an increasingly AI-skeptical content landscape. The retro/pixel art aesthetic is simultaneously experiencing a cultural renaissance, with active communities on TikTok, Instagram, and YouTube.

**Confidence**: Medium. TikTok policy changes are projected, not confirmed. The cultural trend is well-documented.

**Source**: pixel-art-machinima-puppeteering-brief.md, Section 4.

### 7. Multi-gamepad support (up to 4) is a differentiator with social/collaborative potential

No browser-based creative tool supports simultaneous multi-gamepad input for collaborative content creation. The Gamepad API supports up to 4 controllers, and Phaser's built-in GamepadPlugin handles this natively. Four friends controlling four characters on one screen, speaking dialogue, and recording a scene together is a use case with strong viral potential -- it is essentially a pixel art improv theater session.

**Confidence**: High (technical feasibility). Medium (market reception -- untested concept).

**Source**: browser-game-recording-tech-brief.md, Section 2; intake.md (multi-gamepad specification).

### 8. Background tab throttling is the most dangerous UX pitfall

Chrome reduces requestAnimationFrame to ~1fps when a tab is backgrounded. If a user is recording a 3-minute scene and accidentally clicks a notification or switches tabs, the recording output will be choppy/broken with no way to recover the take. This is not a bug -- it is intended browser behavior for power efficiency. The solution must include: (a) Page Visibility API detection with prominent warnings, (b) auto-pause of recording on tab switch, (c) potentially a "recording in progress" browser notification or fullscreen mode recommendation.

**Confidence**: High. Well-documented browser behavior confirmed across multiple sources.

**Source**: browser-game-recording-tech-brief.md, Section 3 (captureStream gotchas).

---

## Recommendations

### R1. Record directly to MP4/H.264 on Chrome/Edge as the primary path
**Rationale**: Chrome 126+ supports `video/mp4; codecs="avc1.42E01E"` in MediaRecorder. This produces files immediately compatible with TikTok, YouTube Shorts, and Instagram Reels without any transcoding step. Reserve ffmpeg.wasm as a lazy-loaded fallback for Firefox users only. This eliminates the 22MB WASM bundle for the majority of users and removes the most complex part of the originally-specified recording pipeline.

### R2. Start with Phaser 3.90.0; architect for v4 migration
**Rationale**: v3.90.0 is the final, most stable v3 release with a mature ecosystem of Vue templates, plugins, and community resources. The EventBridge pattern (mitt) decouples Vue from Phaser internals, minimizing migration surface. Plan v4 migration for post-MVP once the Vue template and plugin ecosystem confirms v4 compatibility (estimated Q3-Q4 2026). Avoid Phavuer -- its tight coupling to Phaser 3 objects would make migration harder.

### R3. Adopt LPC 64x64 as the default sprite format with Aseprite JSON import
**Rationale**: LPC is the closest thing to a sprite sheet standard, with extensive free assets and a browser-based character generator. Aseprite PNG+JSON is the de facto export format used by the pixel art community. Supporting both formats maximizes the available asset pool and aligns with creator workflows. Allow configurable frame sizes (32x32, 64x64) to support non-LPC assets from itch.io.

### R4. Implement aggressive tab-visibility protection for the recording pipeline
**Rationale**: Background tab throttling will silently destroy recordings with no recovery path. Implement: Page Visibility API monitoring, auto-pause on tab switch with visual/audio indicator, "recording active" badge in the UI, and a recommendation to use fullscreen mode during recording. Consider offering `getDisplayMedia()` as an alternative capture method for users who need to switch tabs (though this adds UX complexity).

### R5. Position multi-gamepad collaborative recording as the viral growth mechanism
**Rationale**: Four friends, four gamepads, four characters, one screen, one recording. This is the "party game" moment for content creation -- it is inherently shareable, inherently social, and has no equivalent in any existing tool. Feature it prominently in marketing and onboarding, even if most initial usage is single-player. The collaborative recording use case is the strongest viral loop available.

### R6. Build the codec detection and format negotiation layer early
**Rationale**: Cross-browser codec support varies dramatically. The recording pipeline must use `MediaRecorder.isTypeSupported()` to negotiate the best available format at runtime. Build this as a first-class service (not an afterthought) with a clear fallback chain: MP4/H.264 -> WebM/VP9 -> WebM/VP8 -> error with guidance. Surface the recording format to the user so they know what they are getting.

### R7. Monitor veadotube full version and s&box launch
**Rationale**: veadotube's full version (2026) could add scene/stage features, and s&box (April 28, 2026) could attract machinima creators with modern Source 2 tooling. Neither directly threatens Virtual Stage's niche today, but both could shift the competitive landscape. Set up quarterly competitive monitoring.

### R8. Design for the "authenticity premium" in content metadata and branding
**Rationale**: As AI content faces increasing platform scrutiny, hand-controlled pixel art performances have an inherent authenticity advantage. Consider embedding metadata in exported videos indicating "performed live with gamepad" (similar to how some platforms flag AI-generated content). This is a branding opportunity: Virtual Stage content is "live-acted pixel art" -- a fundamentally human creative act.

---

## Sources

- **browser-game-recording-tech-brief.md** (domain: browser-game-recording-tech, updated 2026-04-12, confidence: medium-high): Phaser v3/v4 ecosystem, Gamepad API compatibility, MediaRecorder codec support, canvas.captureStream() limitations, ffmpeg.wasm performance, Vue 3 + Phaser integration patterns.
- **pixel-art-machinima-puppeteering-brief.md** (domain: pixel-art-machinima-puppeteering, updated 2026-04-12, confidence: medium-high): Machinima tool landscape, VTuber/puppet tool ecosystem, pixel art animation tools, content creation market sizing, open-license asset systems, competitive gap analysis.
- **intake.md** (Virtual Stage project intake): Project classification, vision, requirements, tech stack decisions, development phases, competitive context.

### Key External References (via knowledge briefs)
- Phaser 4.0.0 release: phaser.io/download/release/v4.0.0
- Phaser 3.90.0 final release: phaser.io/news/2025/05/phaser-v390-released
- VTuber market sizing: streammetrix.com/blog/2026-vtuber-evolution
- veadotube mini: veado.tube, olmewe.itch.io/veadotube-mini
- Universal LPC Spritesheet Generator: liberatedpixelcup.github.io
- Chrome MediaRecorder codec support: developer.chrome.com/blog/chrome-136-beta
- W3C Gamepad Specification: w3.org/TR/gamepad/
- phaser3-merged-input: github.com/GaryStanton/phaser3-merged-input
- Aseprite v1.3.17: blog.aseprite.org/2026/02/25/aseprite-v1317/
- iClone Motion Director / Video Mocap: magazine.reallusion.com
- s&box launch: April 28, 2026 on Steam
