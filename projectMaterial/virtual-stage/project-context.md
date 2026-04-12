# Virtual Stage: 2D Pixel Art Scene Acting Tool

## Project Vision

**Virtual Stage** is a browser-based creative tool that turns gamepads into puppet controllers for pixel art characters. Users control characters in a 2D pixel art environment using gamepad joysticks, speak dialogue through microphones, and record the resulting scenes as downloadable video files.

Think of it as a **machinima tool for pixel art** — a virtual puppet theater where anyone can direct, act, and produce animated scenes without animation skills, expensive software, or 3D engine knowledge.

---

## Problem Statement

Content creators, streamers, and indie filmmakers who want to produce pixel-art animated scenes with voice acting currently have no accessible tool to do so. Existing options:

- **3D machinima tools** (Source Filmmaker, Unreal Engine) require deep technical skill and powerful hardware
- **Traditional animation software** (Aseprite, Spine) requires frame-by-frame animation expertise
- **Game engines** (Unity, Godot) require programming knowledge to set up character controllers
- **AI video generators** produce generic output with no real-time creative control

There is no tool that lets someone pick up a gamepad, control a pixel art character in real-time, speak dialogue, and hit record. Virtual Stage fills this gap.

---

## Target Users

### Primary: Content Creators
- YouTubers and TikTokers who create gaming-adjacent content
- Want to produce short animated skits (30s-5min) quickly
- Familiar with gamepads from gaming
- Value speed of production over polish
- Typical workflow: pick characters, set a scene, improvise dialogue, record, upload

### Secondary: Streamers
- Twitch/YouTube Live streamers who use virtual avatars
- Want real-time pixel art puppets controlled by gamepad during streams
- Need low-latency character response to controller input
- May use as an overlay or scene element in OBS/Streamlabs

### Tertiary: Indie Filmmakers
- Indie game developers and animation hobbyists
- Produce longer-form animated shorts (5-30min)
- Need multi-character scenes with precise camera control
- May do multiple takes and edit externally

### Persona: "The Puppeteer"
- Age 18-35, plays video games regularly, owns at least one gamepad
- Has ideas for animated content but lacks animation skills
- Comfortable with browser-based tools
- Wants the creative control of real-time performance, not pre-scripted animation
- Values expressiveness: "I want the character to feel like I'm controlling it live"

---

## Core Requirements

### R1: Gamepad-Controlled Character Movement
- Left joystick controls character movement (walk/run based on stick magnitude)
- Right joystick controls look-at direction (character faces independently of movement direction)
- Deadzone handling (0.15 threshold) to prevent stick drift
- Support for standard gamepad layout (Xbox/PlayStation/Switch via Gamepad API standard mapping)
- Smooth, responsive character movement at 30-60fps

### R2: Theatrical Action System
- All actions are theatrical/expressive — no game mechanics, no hitboxes, no damage
- Actions triggered by gamepad buttons with intuitive mapping
- Action categories:
  - **Movement**: idle, walk, run, jump, crouch
  - **Combat (theatrical)**: sword swing/punch, magic/ranged, shield block
  - **Emotes**: wave, nod, shrug, laugh, cry, point, celebrate, sit down, fall down
  - **Special**: theatrical death, get up
- Emote wheel system: bumper buttons (LB/RB) as modifiers + face buttons = 8 emotes from 4 buttons
- Character state machine: IDLE, WALK, RUN, JUMP, CROUCH, ACTION, EMOTE, DEAD
- Non-interruptible actions play to completion; hold actions release on button release
- Look-at direction is always processed regardless of current action state

### R3: Microphone Input
- Capture microphone audio via getUserMedia API
- Gain control (volume slider) via Web Audio API GainNode
- Visual audio level indicator so user knows mic is active
- Audio stream feeds directly into the recording pipeline
- No echo/feedback — user hears themselves naturally, not through the app

### R4: Video Recording
- Record canvas output + microphone audio as a single video file
- Canvas captured via captureStream(30) at 30fps
- Audio and video combined into a single MediaStream
- MediaRecorder produces WebM (VP9+Opus) output
- Recording flow: Start (with 3-2-1 countdown) -> Record -> Stop -> Preview -> Download/Discard
- Gamepad Start button triggers recording start/stop
- Select button hides Vue UI overlay for clean recording frame
- Preview modal plays back the recording before user commits to saving
- Download as WebM file

### R5: Multi-Character Support
- Up to 4 characters in a single scene
- Each character assignable to a different gamepad (Gamepad API supports 4 pads)
- Multiple people can control different characters simultaneously (the ideal use case)
- Single user can switch active character when using one gamepad
- Non-active characters hold their last pose (idle animation)

### R6: Scene Management
- Background selection from a library of pixel art environments
- Camera modes: Fixed (full stage), Follow (tracks active character), Free Pan (manual control)
- Props: static sprites placed in scene via UI, z-ordered by Y position
- Scene serialization to JSON for save/load
- Local save (localStorage + file export) and cloud save (Firestore)

### R7: Asset System
- Characters as sprite sheets with JSON animation definitions
- 4-directional sprites (down, up, left, right) with horizontal flip for left/right
- Initial asset set sourced from open-license pixel art packs (itch.io)
- Asset manifest system for loading and caching
- Support for multiple character types (knight, mage, rogue, etc.) with different action animations

---

## Gamepad Button Mapping

```
Left Stick  = Movement (magnitude determines walk/run speed)
Right Stick = Look-at direction (character faces this direction independently)
A / Cross   = Jump
B / Circle  = Context action / Nod
X / Square  = Primary attack (theatrical sword swing / punch)
Y / Triangle = Magic / Ranged attack (theatrical)
LB / L1 + face button = Emote wheel 1 (LB+A=Wave, LB+B=Shrug, LB+X=Laugh, LB+Y=Cry)
RB / R1 + face button = Emote wheel 2 (RB+A=Celebrate, RB+B=Point, RB+X=Sit, RB+Y=Fall)
LT / L2 (hold) = Crouch/Duck
RT / R2 (hold) = Shield Block / Defend
D-pad Up    = Cycle character costume
D-pad Down  = Theatrical death animation
L3 (left stick click) = Toggle run lock
R3 (right stick click) = Reset facing to movement direction
Select/Back = Toggle UI overlay visibility
Start       = Start/Stop recording
```

The emote wheel pattern (bumper + face button) is proven in games like Dark Souls, Fortnite, and Elden Ring. It's intuitive for any gamepad user.

---

## Technical Architecture

### Tech Stack Decision

| Layer | Choice | Rationale |
|-------|--------|-----------|
| UI Framework | Vue 3 (Composition API + TypeScript) | Team's established framework |
| 2D Game Engine | Phaser.js v3 | Best-in-class browser 2D engine: native gamepad plugin, sprite animation manager, camera system, tilemap support. Official Vue 3 template. Largest ecosystem (~3.5K GitHub stars, 100K+ monthly npm downloads) |
| Build Tool | Vite | Default for Vue 3, fast HMR, excellent Phaser compatibility |
| Vue-Phaser Bridge | mitt (200-byte EventEmitter) | Framework-agnostic typed event bus. Neither Vue nor Phaser reaches into the other's state |
| Video Recording | MediaRecorder API (browser-native) | captureStream + getUserMedia. Zero server dependencies |
| Audio Capture | Web Audio API + getUserMedia | Browser-native mic access with gain control |
| Backend | Firebase (Auth + Firestore + Cloud Storage) | Cloud persistence for scenes and recordings |
| Language | TypeScript | Type safety across Vue/Phaser/Services boundary |

### Why Phaser.js Over Alternatives

**vs. PixiJS**: PixiJS is purely a renderer (~160KB). No built-in animation system, gamepad support, camera system, or tilemap support. Would require building all of these from scratch — weeks of extra work. Phaser provides all of them out of the box (~1MB bundle, acceptable tradeoff).

**vs. Excalibur.js**: Smaller community, less documentation, ECS paradigm less familiar. Phaser's imperative API is more accessible.

**vs. Kaboom.js/Kaplay**: Too lightweight for production. Better for game jams. Uncertain long-term maintenance.

**vs. Raw Canvas API**: Maximum control but reinvents every wheel. 2-4x development time. Not justified.

### Architecture Diagram

```
Browser Window
+--------------------------------------------------+
|  Vue 3 UI Shell          |  Phaser.js Canvas      |
|  - TopBar (rec status)   |  - StageScene          |
|  - CharacterPanel        |    - Background         |
|  - ScenePanel            |    - Characters         |
|  - RecordingControls     |    - Props              |
|  - ActionBindingEditor   |    - Camera             |
|  - PreviewModal          |                         |
|          |               |                         |
|          +-- EventBridge (mitt) --+                |
+--------------------------------------------------+
|              Shared Services                      |
|  GamepadService | RecordingService | AudioService |
+--------------------------------------------------+
|              Firebase Backend (optional)           |
|  Auth | Firestore (scenes) | Storage (videos)     |
+--------------------------------------------------+
```

**Key architectural boundary**: Phaser owns the canvas and all real-time game logic (rendering, animation, movement). Vue owns the DOM overlay and all UI (menus, panels, modals, recording controls). They communicate through a shared EventBridge (mitt) — a plain TypeScript class that both sides import. Neither framework reaches into the other's state.

### Project Structure

```
src/
  bridge/
    EventBridge.ts            # typed event emitter
    types.ts                  # shared event type definitions
  phaser/
    PhaserGame.ts             # Phaser.Game factory + config
    scenes/
      StageScene.ts           # main scene: background + characters + props
      PreloaderScene.ts       # asset loading with progress bar
    entities/
      Character.ts            # sprite + state machine + animation controller
      Prop.ts                 # static/animated scene prop
    systems/
      CharacterController.ts  # reads gamepad -> drives character
      CameraController.ts     # camera follow/pan/fixed modes
      ActionSystem.ts         # action dispatch + animation triggers
  vue/
    App.vue
    components/
      PhaserCanvas.vue        # mounts/destroys Phaser instance
      RecordingControls.vue   # start/stop/preview recording
      CharacterPanel.vue      # select character, assign gamepad
      ScenePanel.vue          # backgrounds, props palette
      ActionBindingEditor.vue # remap gamepad buttons to actions
      PreviewModal.vue        # playback recorded video
      TopBar.vue              # scene name, recording indicator, timer
  services/
    GamepadService.ts         # polls navigator.getGamepads(), normalizes input
    RecordingService.ts       # captureStream + MediaRecorder pipeline
    AudioService.ts           # getUserMedia, gain control
    AssetService.ts           # sprite sheet loading, manifest resolution
    SceneStateService.ts      # serialize/deserialize scene state JSON
  assets/
    characters/               # sprite sheets (.png) + animation JSON
    backgrounds/              # scene background images
    props/                    # prop sprite sheets
```

### Recording Pipeline

```
Phaser Canvas                  Microphone
captureStream(30)              getUserMedia({ audio: true })
     |                              |
     +----> MediaStream <-----------+
               (combined video + audio tracks)
                    |
              MediaRecorder
              (webm; vp9+opus, 2.5Mbps video)
                    |
              Blob chunks -> final Blob -> Preview / Download
```

Audio goes through a Web Audio API gain node before reaching the MediaRecorder, allowing mic volume control:

```
getUserMedia -> MediaStreamSource -> GainNode -> MediaStreamDestination
                                       ^
                                  (mic volume slider in Vue UI)
```

### Standalone GamepadService (not Phaser's plugin)

Phaser has a built-in gamepad plugin, but we build a standalone GamepadService because:
1. Vue components need gamepad state too (ActionBindingEditor needs to detect button presses for remapping)
2. Recording controls need gamepad input (Start button)
3. The service can be unit-tested independently of Phaser
4. It avoids coupling to Phaser's lifecycle

The service polls `navigator.getGamepads()` each frame, applies deadzone filtering (0.15), debounces button presses, and tracks up to 4 gamepads independently.

---

## Scene State Format

Scenes are serialized as JSON for save/load:

```json
{
  "id": "scene-uuid",
  "name": "The Betrayal",
  "background": "castle-hall",
  "camera": { "mode": "fixed", "zoom": 1.0, "x": 0, "y": 0 },
  "characters": [
    {
      "id": "char-1",
      "spriteSheet": "knight",
      "position": { "x": 400, "y": 900 },
      "facing": "right",
      "costume": "default",
      "gamepadIndex": 0
    }
  ],
  "props": [
    {
      "id": "prop-1",
      "sprite": "throne",
      "position": { "x": 600, "y": 850 },
      "scale": 1.0,
      "layer": "background",
      "flipX": false
    }
  ]
}
```

---

## Development Phases

### Phase 1: Foundation
Phaser renders a character on a background, controllable with gamepad left stick (movement) and right stick (facing). Basic Vue shell.

### Phase 2: Actions + Audio
Full action state machine with all theatrical actions. Emote wheel. Mic capture with gain control.

### Phase 3: Recording Pipeline (MVP)
Record canvas + mic as WebM video. Countdown, preview, download. **This is the minimum viable product.**

### Phase 4: Multi-Character + Scenes
Multiple gamepads controlling multiple characters. Background selector. Camera modes. Scene save/load.

### Phase 5: Props + Polish
Prop placement, speech bubbles, scene transitions, keyboard fallback, onboarding.

### Phase 6: Firebase + Cloud
User accounts, cloud scene persistence, recording storage, shareable links.

---

## Platform & Browser Target

- **Platform**: Browser-only (no Electron, no mobile)
- **Primary browsers**: Chrome, Edge (Chromium-based)
- **Secondary**: Firefox (Gamepad API and MediaRecorder work but have quirks)
- **Not targeted**: Safari (poor MediaRecorder and Gamepad API support)

---

## Art Assets Strategy

- **Initial development**: Source from open-license pixel art packs on itch.io
- **Needed for MVP**: 1 character (knight) with full animation set, 1 background, basic UI
- **Full asset set**: 3 character types, 5 backgrounds, 15 props
- **Future**: Custom asset uploads by users, community asset sharing

---

## Competitive Landscape

There is no direct competitor offering this exact combination (gamepad + pixel art + mic + recording). Adjacent tools include:

- **Source Filmmaker / Garry's Mod**: 3D machinima, complex, requires gaming PC
- **VTuber software** (VSeeFace, VTube Studio): Face-tracking avatars, not pixel art, not gamepad-controlled
- **Puppet Pals / Toontastic**: Mobile puppet theater apps for kids, not pixel art, no gamepad
- **RPG Maker**: Game creation, not scene acting/recording
- **Aseprite**: Pixel art animation tool, frame-by-frame (not real-time)

Virtual Stage occupies a unique niche: **real-time, gamepad-controlled, pixel art puppeteering with voice recording**.

---

## Key Risks

1. **Gamepad API browser inconsistency**: Different controllers report axes/buttons differently. Mitigation: extensive mapping testing, configurable bindings.
2. **MediaRecorder codec support**: VP9+Opus not universal. Mitigation: feature-detect and fallback to VP8+Opus.
3. **Art asset availability**: Open-license pixel art with full animation sets (4 directions, multiple actions) is limited. Mitigation: placeholder art during development, commission original art later.
4. **Vue-Phaser lifecycle conflicts**: Phaser's game loop and Vue's reactivity can conflict. Mitigation: strict EventBridge boundary, no shared state objects.
5. **Recording memory usage**: Long recordings accumulate large Blob arrays in memory. Mitigation: chunk flushing, reasonable recording limits (10-15 min).
