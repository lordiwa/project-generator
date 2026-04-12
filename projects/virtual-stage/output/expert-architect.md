# Expert Analysis: Architect — Virtual Stage

**Project**: Virtual Stage
**Date**: 2026-04-12
**Agent**: Software Architect
**Input Sources**: intake.md, expert-business-analyst.md, expert-research-analyst.md, browser-game-recording-tech-brief.md, pixel-art-machinima-puppeteering-brief.md

---

## Architecture Overview

Virtual Stage is a browser-based creative tool built as a **dual-framework application**: a Vue 3 shell providing UI overlays (menus, panels, recording controls) and a Phaser 3.90.0 canvas rendering the pixel art stage. The two frameworks share no state directly. All communication flows through a typed **EventBridge** (mitt), and cross-cutting services (GamepadService, RecordingService, AudioService, SceneStateService, AssetService) live in a neutral service layer accessible to both frameworks.

The application is **local-first** — it runs entirely in the browser with no server dependency for MVP. Firebase provides an optional cloud layer (auth, scene sync, recording storage) added in Phase 6. Asset loading, scene serialization, and recording all operate against browser-local APIs (localStorage, IndexedDB, Blob URLs, File API).

### System Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  BROWSER TAB                                                                │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  VUE 3 SHELL (Composition API + TypeScript)                         │   │
│  │  ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌──────────┐ ┌────────┐  │   │
│  │  │ Top Bar  │ │Character │ │  Scene     │ │Recording │ │Preview │  │   │
│  │  │          │ │  Panel   │ │  Panel     │ │ Controls │ │ Modal  │  │   │
│  │  └──────────┘ └──────────┘ └────────────┘ └──────────┘ └────────┘  │   │
│  │  ┌──────────┐ ┌──────────┐ ┌────────────┐                          │   │
│  │  │ Audio    │ │ Binding  │ │ Onboarding │                          │   │
│  │  │ Controls │ │ Editor   │ │ Overlay    │                          │   │
│  │  └──────────┘ └──────────┘ └────────────┘                          │   │
│  └─────────────────────────┬────────────────────────────────────────────┘   │
│                            │                                                │
│                    ┌───────▼───────┐                                        │
│                    │  EVENT BRIDGE │  (mitt — typed events)                  │
│                    │  ◄──────────► │                                        │
│                    └───────┬───────┘                                        │
│                            │                                                │
│  ┌─────────────────────────▼────────────────────────────────────────────┐   │
│  │  PHASER 3.90.0 CANVAS (WebGL)                                       │   │
│  │  ┌──────────────────────────────────────────────────────────────┐    │   │
│  │  │  StageScene                                                  │    │   │
│  │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────────────────┐  │    │   │
│  │  │  │Character│ │Character│ │Character│ │  Background       │  │    │   │
│  │  │  │Entity 0 │ │Entity 1 │ │Entity 2 │ │  + Props Layer    │  │    │   │
│  │  │  └─────────┘ └─────────┘ └─────────┘ └───────────────────┘  │    │   │
│  │  │  ┌────────────────────┐ ┌─────────────────────────────────┐  │    │   │
│  │  │  │CharacterController │ │  CameraController               │  │    │   │
│  │  │  │       System       │ │  (Fixed / Follow / FreePan)     │  │    │   │
│  │  │  └────────────────────┘ └─────────────────────────────────┘  │    │   │
│  │  └──────────────────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  SHARED SERVICES (framework-agnostic TypeScript)                     │   │
│  │  ┌──────────────┐ ┌────────────────┐ ┌──────────────────────────┐   │   │
│  │  │GamepadService│ │RecordingService│ │      AudioService        │   │   │
│  │  │(polling loop)│ │(MediaRecorder) │ │ (getUserMedia + GainNode)│   │   │
│  │  └──────────────┘ └────────────────┘ └──────────────────────────┘   │   │
│  │  ┌──────────────────┐ ┌────────────────┐                            │   │
│  │  │SceneStateService │ │  AssetService  │                            │   │
│  │  │(JSON serialize)  │ │(manifest+load) │                            │   │
│  │  └──────────────────┘ └────────────────┘                            │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  FIREBASE (optional cloud layer — Phase 6)                           │   │
│  │  ┌──────────────┐ ┌────────────────┐ ┌──────────────────────────┐   │   │
│  │  │  Firebase    │ │   Firestore    │ │   Cloud Storage          │   │
│  │  │  Auth        │ │  (scene docs)  │ │  (recordings + assets)   │   │
│  │  └──────────────┘ └────────────────┘ └──────────────────────────┘   │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Directory Structure

```
src/
├── main.ts                        # Vue app entry point
├── App.vue                        # Root Vue component
├── bridge/
│   └── EventBridge.ts             # Typed mitt instance + event definitions
├── services/
│   ├── GamepadService.ts          # Standalone gamepad polling + deadzone
│   ├── RecordingService.ts        # captureStream + MediaRecorder pipeline
│   ├── AudioService.ts            # getUserMedia + GainNode + AnalyserNode
│   ├── SceneStateService.ts       # JSON serialization + localStorage + file I/O
│   └── AssetService.ts            # Asset manifest + Phaser loader integration
├── vue/
│   ├── components/
│   │   ├── PhaserCanvas.vue       # Mounts/destroys Phaser game instance
│   │   ├── TopBar.vue             # Scene name, recording indicator, timer
│   │   ├── CharacterPanel.vue     # Character selection, gamepad assignment
│   │   ├── ScenePanel.vue         # Background + props browser
│   │   ├── RecordingControls.vue  # Start/stop, countdown, format indicator
│   │   ├── PreviewModal.vue       # Video playback after recording
│   │   ├── AudioControls.vue      # Gain slider, VU meter
│   │   ├── BindingEditor.vue      # Gamepad remapping UI
│   │   └── OnboardingOverlay.vue  # First-time tutorial
│   ├── composables/
│   │   ├── useGamepad.ts          # Vue composable wrapping GamepadService
│   │   ├── useRecording.ts        # Vue composable wrapping RecordingService
│   │   └── useAudio.ts            # Vue composable wrapping AudioService
│   └── stores/
│       └── uiStore.ts             # Pinia store for UI-only state (panel visibility, etc.)
├── phaser/
│   ├── PhaserGame.ts              # Phaser.Game factory + config
│   ├── scenes/
│   │   ├── BootScene.ts           # Asset preloading + manifest
│   │   ├── StageScene.ts          # Main performance scene
│   │   └── CountdownScene.ts      # 3-2-1 overlay before recording
│   ├── entities/
│   │   └── Character.ts           # Sprite + state machine + animation controller
│   ├── systems/
│   │   ├── CharacterController.ts # Maps gamepad input → character methods
│   │   └── CameraController.ts    # Fixed / Follow / FreePan modes
│   └── utils/
│       └── DepthSort.ts           # Y-based depth sorting for characters + props
├── types/
│   ├── events.ts                  # EventBridge event map type
│   ├── gamepad.ts                 # Gamepad-related types
│   ├── scene.ts                   # Scene state schema types
│   ├── character.ts               # Character definition types
│   ├── assets.ts                  # Asset manifest types
│   └── recording.ts               # Recording metadata types
└── assets/
    ├── manifest.json              # Asset manifest (characters, backgrounds, props)
    ├── characters/                # Sprite sheets + JSON animation defs
    ├── backgrounds/               # Background images
    └── props/                     # Prop sprite sheets
```

### Architectural Principles

1. **Strict framework boundary**: Vue code never imports from `src/phaser/`. Phaser code never imports from `src/vue/`. Both may import from `src/bridge/`, `src/services/`, and `src/types/`.
2. **Events over shared state**: The EventBridge is the only communication channel between Vue and Phaser. No shared Pinia store, no global mutable objects.
3. **Services are framework-agnostic**: All services in `src/services/` are plain TypeScript classes with no Vue or Phaser dependencies. They communicate via callbacks or the EventBridge.
4. **Local-first, cloud-optional**: Every feature works without Firebase. Cloud features are additive, never blocking.
5. **Data-driven assets**: New characters, backgrounds, and props are added by editing the manifest and dropping files — no code changes required.

---

## Components

### PhaserCanvas.vue

- **Responsibility**: Vue component that creates, mounts, and destroys the Phaser.Game instance. Provides the DOM container for the Phaser canvas. Handles lifecycle synchronization between Vue and Phaser.
- **Technology**: Vue 3 Composition API, Phaser 3.90.0
- **Interfaces**:
  - **Props**: `width: number`, `height: number` (canvas dimensions, default 1280x720)
  - **Emits**: `game-ready(game: Phaser.Game)` (one-time, after Phaser boot)
  - **Lifecycle**: `onMounted` creates Phaser.Game; `onBeforeUnmount` calls `game.destroy(true)`
- **Key Implementation Detail**: The Phaser game reference is stored with `shallowRef` (not `ref`) to prevent Vue's reactivity proxy from wrapping Phaser internals. The game instance is also registered with `markRaw`.

```typescript
// PhaserCanvas.vue — setup (simplified)
import { shallowRef, onMounted, onBeforeUnmount, markRaw } from 'vue';
import { createPhaserGame } from '@/phaser/PhaserGame';

const containerRef = ref<HTMLDivElement>();
const gameRef = shallowRef<Phaser.Game | null>(null);

onMounted(() => {
  const game = markRaw(createPhaserGame(containerRef.value!, props.width, props.height));
  gameRef.value = game;
  game.events.once('ready', () => emit('game-ready', game));
});

onBeforeUnmount(() => {
  gameRef.value?.destroy(true);
  gameRef.value = null;
});
```

### EventBridge

- **Responsibility**: Typed event bus connecting Vue and Phaser. Single source of truth for all cross-framework communication. Every event has a defined payload type checked at compile time.
- **Technology**: mitt v3.x
- **Interfaces**: See full event map below.

```typescript
// src/bridge/EventBridge.ts
import mitt from 'mitt';
import type { EventMap } from '@/types/events';

export const eventBridge = mitt<EventMap>();

// src/types/events.ts
import type { CharacterState, Direction } from './character';
import type { CameraMode } from './scene';
import type { RecordingFormat } from './recording';
import type { GamepadInput } from './gamepad';

export type EventMap = {
  // === Vue → Phaser ===
  'scene:set-background':       { backgroundId: string };
  'scene:add-character':        { characterId: string; slotIndex: number };
  'scene:remove-character':     { slotIndex: number };
  'scene:add-prop':             { propId: string; x: number; y: number };
  'scene:remove-prop':          { propInstanceId: string };
  'scene:update-prop':          { propInstanceId: string; x?: number; y?: number; scaleX?: number; scaleY?: number; flipX?: boolean; layer?: 'foreground' | 'background' };
  'scene:load-state':           { state: SceneState };
  'camera:set-mode':            { mode: CameraMode; targetSlot?: number };
  'camera:set-zoom':            { zoom: number };
  'character:set-costume':      { slotIndex: number; costumeId: string };
  'character:assign-gamepad':   { slotIndex: number; gamepadIndex: number | null };
  'recording:countdown-start':  void;
  'recording:countdown-tick':   { remaining: number };
  'recording:started':          void;
  'recording:stopped':          void;
  'ui:set-visibility':          { visible: boolean };

  // === Phaser → Vue ===
  'phaser:ready':               void;
  'phaser:scene-loaded':        { backgroundId: string };
  'character:state-changed':    { slotIndex: number; state: CharacterState };
  'character:position-changed': { slotIndex: number; x: number; y: number; direction: Direction };
  'camera:mode-changed':        { mode: CameraMode };
  'scene:state-snapshot':       { state: SceneState };

  // === Service → Any ===
  'gamepad:connected':          { index: number; id: string };
  'gamepad:disconnected':       { index: number };
  'gamepad:input':              { gamepadIndex: number; input: GamepadInput };
  'recording:error':            { message: string };
  'recording:memory-warning':   { usedMB: number; limitMB: number };
  'recording:visibility-lost':  void;
  'recording:visibility-restored': void;
  'audio:level':                { peak: number; rms: number };
  'audio:permission-denied':    void;
  'asset:load-progress':        { loaded: number; total: number };
  'asset:load-complete':        void;
  'asset:load-error':           { assetId: string; error: string };
};
```

### GamepadService

- **Responsibility**: Framework-agnostic service that polls the Gamepad API every frame, applies radial deadzone filtering, tracks button state transitions (pressed/released/held), debounces inputs, and emits normalized input events. Supports up to 4 simultaneous gamepads. Operates outside Phaser's game loop so Vue components can also read gamepad state.
- **Technology**: Gamepad API, requestAnimationFrame
- **Interfaces**: EventBridge events (`gamepad:connected`, `gamepad:disconnected`, `gamepad:input`), direct state query via `getState(index)`

```typescript
// src/services/GamepadService.ts

export interface StickInput {
  x: number;     // -1 to 1 after deadzone
  y: number;     // -1 to 1 after deadzone
  magnitude: number; // 0 to 1
  angle: number;     // radians
}

export interface GamepadInput {
  leftStick: StickInput;
  rightStick: StickInput;
  buttons: ButtonState[];
  timestamp: number;
}

export interface ButtonState {
  index: number;
  pressed: boolean;       // true on the frame the button was pressed
  released: boolean;      // true on the frame the button was released
  held: boolean;          // true while the button is physically down
  value: number;          // 0-1 for analog triggers
}

export interface GamepadServiceConfig {
  deadzone: number;               // default 0.15
  deadzonePerPad?: Map<number, number>; // per-gamepad override
  pollRate: 'raf' | number;       // 'raf' = every animation frame, number = ms interval
  debounceMs: number;             // default 100ms for button presses
}

export class GamepadService {
  private config: GamepadServiceConfig;
  private padStates: Map<number, GamepadInput> = new Map();
  private prevRawButtons: Map<number, boolean[]> = new Map();
  private lastButtonTime: Map<string, number> = new Map(); // "padIndex:btnIndex" -> timestamp
  private rafId: number | null = null;
  private running = false;

  constructor(config?: Partial<GamepadServiceConfig>) {
    this.config = {
      deadzone: 0.15,
      pollRate: 'raf',
      debounceMs: 100,
      ...config,
    };
  }

  start(): void {
    if (this.running) return;
    this.running = true;
    window.addEventListener('gamepadconnected', this.onConnect);
    window.addEventListener('gamepaddisconnected', this.onDisconnect);
    this.poll();
  }

  stop(): void {
    this.running = false;
    if (this.rafId !== null) cancelAnimationFrame(this.rafId);
    window.removeEventListener('gamepadconnected', this.onConnect);
    window.removeEventListener('gamepaddisconnected', this.onDisconnect);
  }

  getState(gamepadIndex: number): GamepadInput | null {
    return this.padStates.get(gamepadIndex) ?? null;
  }

  getConnectedPads(): number[] {
    return [...this.padStates.keys()];
  }

  setDeadzone(gamepadIndex: number, deadzone: number): void {
    if (!this.config.deadzonePerPad) this.config.deadzonePerPad = new Map();
    this.config.deadzonePerPad.set(gamepadIndex, deadzone);
  }

  setGlobalDeadzone(deadzone: number): void {
    this.config.deadzone = deadzone;
  }

  private poll = (): void => {
    if (!this.running) return;
    const gamepads = navigator.getGamepads();
    const now = performance.now();

    for (let i = 0; i < gamepads.length; i++) {
      const gp = gamepads[i];
      if (!gp || !gp.connected) continue;

      const dz = this.config.deadzonePerPad?.get(i) ?? this.config.deadzone;
      const leftStick = this.applyRadialDeadzone(gp.axes[0], gp.axes[1], dz);
      const rightStick = this.applyRadialDeadzone(gp.axes[2], gp.axes[3], dz);

      const prevButtons = this.prevRawButtons.get(i) ?? [];
      const buttons: ButtonState[] = gp.buttons.map((btn, idx) => {
        const wasHeld = prevButtons[idx] ?? false;
        const isHeld = btn.pressed;
        const key = `${i}:${idx}`;
        const lastTime = this.lastButtonTime.get(key) ?? 0;
        const debounced = (now - lastTime) < this.config.debounceMs;

        let pressed = false;
        let released = false;

        if (isHeld && !wasHeld && !debounced) {
          pressed = true;
          this.lastButtonTime.set(key, now);
        }
        if (!isHeld && wasHeld) {
          released = true;
        }

        return { index: idx, pressed, released, held: isHeld, value: btn.value };
      });

      this.prevRawButtons.set(i, gp.buttons.map(b => b.pressed));

      const input: GamepadInput = {
        leftStick,
        rightStick,
        buttons,
        timestamp: now,
      };
      this.padStates.set(i, input);
      eventBridge.emit('gamepad:input', { gamepadIndex: i, input });
    }

    this.rafId = requestAnimationFrame(this.poll);
  };

  private applyRadialDeadzone(rawX: number, rawY: number, deadzone: number): StickInput {
    const magnitude = Math.sqrt(rawX * rawX + rawY * rawY);
    if (magnitude < deadzone) {
      return { x: 0, y: 0, magnitude: 0, angle: 0 };
    }
    const normalizedX = rawX / magnitude;
    const normalizedY = rawY / magnitude;
    const scaledMagnitude = Math.min((magnitude - deadzone) / (1 - deadzone), 1);
    return {
      x: normalizedX * scaledMagnitude,
      y: normalizedY * scaledMagnitude,
      magnitude: scaledMagnitude,
      angle: Math.atan2(normalizedY, normalizedX),
    };
  }

  private onConnect = (e: GamepadEvent): void => {
    eventBridge.emit('gamepad:connected', { index: e.gamepad.index, id: e.gamepad.id });
  };

  private onDisconnect = (e: GamepadEvent): void => {
    this.padStates.delete(e.gamepad.index);
    this.prevRawButtons.delete(e.gamepad.index);
    eventBridge.emit('gamepad:disconnected', { index: e.gamepad.index });
  };
}
```

### Character Entity

- **Responsibility**: Encapsulates a single character on the stage: its Phaser sprite, animation state machine, facing direction, current position, and animation controller. Exposes methods for state transitions and animation playback. Does not know about gamepads or input — it receives commands.
- **Technology**: Phaser 3.90.0 Sprite, custom state machine
- **Interfaces**: Methods called by CharacterController; emits events via EventBridge for state/position changes.

```typescript
// src/phaser/entities/Character.ts

export enum CharacterState {
  IDLE = 'IDLE',
  WALK = 'WALK',
  RUN = 'RUN',
  JUMP = 'JUMP',
  CROUCH = 'CROUCH',
  ACTION = 'ACTION',
  EMOTE = 'EMOTE',
  DEAD = 'DEAD',
}

export enum Direction {
  DOWN = 'down',
  UP = 'up',
  LEFT = 'left',
  RIGHT = 'right',
}

export interface CharacterConfig {
  characterId: string;
  slotIndex: number;
  spriteKey: string;
  animationDefs: AnimationDefinitionMap;
  startX: number;
  startY: number;
  walkSpeed: number;    // pixels per second
  runSpeed: number;     // pixels per second
  walkThreshold: number; // stick magnitude below this = walk, above = run
}

export interface AnimationDefinitionMap {
  idle:    DirectionalAnimation;
  walk:    DirectionalAnimation;
  run:     DirectionalAnimation;
  jump:    DirectionalAnimation;
  crouch:  DirectionalAnimation;
  attack1: DirectionalAnimation;  // sword / punch
  attack2: DirectionalAnimation;  // magic / ranged
  action:  DirectionalAnimation;  // context / nod
  block:   DirectionalAnimation;
  hurt:    DirectionalAnimation;
  death:   Animation;             // single direction
  getUp:   Animation;             // single direction
  emotes:  Record<string, Animation>; // wave, shrug, laugh, cry, etc.
}

export interface DirectionalAnimation {
  [Direction.DOWN]:  Animation;
  [Direction.UP]:    Animation;
  [Direction.LEFT]:  Animation;
  [Direction.RIGHT]: Animation;
}

export interface Animation {
  key: string;           // Phaser animation key
  frameRate: number;     // FPS
  repeat: number;        // -1 = loop, 0 = play once
  interruptible: boolean;
}

export class Character {
  public sprite: Phaser.GameObjects.Sprite;
  public state: CharacterState = CharacterState.IDLE;
  public direction: Direction = Direction.DOWN;
  public lookDirection: Direction = Direction.DOWN;
  public slotIndex: number;
  public characterId: string;

  private config: CharacterConfig;
  private scene: Phaser.Scene;
  private locked = false; // true during non-interruptible animations

  constructor(scene: Phaser.Scene, config: CharacterConfig) { /* ... */ }

  // --- Public methods called by CharacterController ---

  move(dx: number, dy: number, magnitude: number): void {
    // Determines walk vs run from magnitude vs walkThreshold
    // Updates direction from dx/dy
    // Sets velocity on sprite body
    // Transitions to WALK or RUN state
  }

  stopMoving(): void {
    // Zero velocity, transition to IDLE
  }

  setLookDirection(direction: Direction): void {
    // Updates lookDirection independently of movement direction
    // Flips sprite if needed
  }

  jump(): boolean {
    // Returns false if locked (non-interruptible action in progress)
    // Plays jump animation, sets locked = true until animation completes
  }

  attack(type: 'attack1' | 'attack2' | 'action'): boolean { /* ... */ }

  crouch(active: boolean): void {
    // CROUCH state while held, IDLE on release
  }

  block(active: boolean): void {
    // Block pose while held, IDLE on release
  }

  emote(emoteId: string): boolean {
    // Play emote animation, locked until complete
  }

  die(): boolean {
    // Transition to DEAD state, locked until getUp()
  }

  getUp(): boolean {
    // Transition from DEAD to IDLE via getUp animation
  }

  setCostume(costumeId: string): void {
    // Swap sprite texture to alternate costume spritesheet
  }

  // --- State machine ---

  private setState(newState: CharacterState): void {
    if (this.state === newState) return;
    if (this.locked && !this.isTransitionAllowed(this.state, newState)) return;
    this.state = newState;
    this.playAnimation();
    eventBridge.emit('character:state-changed', {
      slotIndex: this.slotIndex,
      state: this.state,
    });
  }

  private isTransitionAllowed(from: CharacterState, to: CharacterState): boolean {
    // DEAD -> only IDLE (via getUp)
    // Locked states (JUMP, ACTION, EMOTE) -> only after animation complete
    // All other transitions allowed
  }

  private playAnimation(): void {
    // Selects animation key based on current state + direction (or lookDirection for idle)
    // Plays via sprite.anims.play()
    // On complete callback: unlock and check for queued transitions
  }

  serialize(): CharacterSnapshot {
    return {
      characterId: this.characterId,
      slotIndex: this.slotIndex,
      x: this.sprite.x,
      y: this.sprite.y,
      direction: this.direction,
      lookDirection: this.lookDirection,
      state: this.state,
      costumeId: this.config.spriteKey,
    };
  }
}
```

### CharacterController System

- **Responsibility**: Listens to gamepad input events from the EventBridge, maps them to Character entity methods based on the current button binding configuration. Handles the emote wheel modifier system (LB/RB + face buttons). Manages gamepad-to-character slot assignments.
- **Technology**: Pure TypeScript, EventBridge listener
- **Interfaces**: Consumes `gamepad:input` events, calls Character methods, reads from ActionBindingConfig.

```typescript
// src/phaser/systems/CharacterController.ts

export interface ActionBinding {
  buttonIndex: number;
  modifier?: 'LB' | 'RB' | null;
  action: string;  // method name or emote ID
}

// Standard Gamepad button indices (W3C Standard Gamepad)
export const STANDARD_BUTTONS = {
  A: 0, B: 1, X: 2, Y: 3,
  LB: 4, RB: 5, LT: 6, RT: 7,
  SELECT: 8, START: 9,
  L3: 10, R3: 11,
  DPAD_UP: 12, DPAD_DOWN: 13, DPAD_LEFT: 14, DPAD_RIGHT: 15,
} as const;

export const DEFAULT_BINDINGS: ActionBinding[] = [
  { buttonIndex: STANDARD_BUTTONS.A, action: 'jump' },
  { buttonIndex: STANDARD_BUTTONS.X, action: 'attack1' },
  { buttonIndex: STANDARD_BUTTONS.Y, action: 'attack2' },
  { buttonIndex: STANDARD_BUTTONS.B, action: 'action' },
  { buttonIndex: STANDARD_BUTTONS.LB, modifier: 'LB', action: 'emote-modifier' },
  { buttonIndex: STANDARD_BUTTONS.RB, modifier: 'RB', action: 'emote-modifier' },
  // LB + face buttons = emote set 1
  { buttonIndex: STANDARD_BUTTONS.A, modifier: 'LB', action: 'emote:wave' },
  { buttonIndex: STANDARD_BUTTONS.B, modifier: 'LB', action: 'emote:shrug' },
  { buttonIndex: STANDARD_BUTTONS.X, modifier: 'LB', action: 'emote:laugh' },
  { buttonIndex: STANDARD_BUTTONS.Y, modifier: 'LB', action: 'emote:cry' },
  // RB + face buttons = emote set 2
  { buttonIndex: STANDARD_BUTTONS.A, modifier: 'RB', action: 'emote:celebrate' },
  { buttonIndex: STANDARD_BUTTONS.B, modifier: 'RB', action: 'emote:point' },
  { buttonIndex: STANDARD_BUTTONS.X, modifier: 'RB', action: 'emote:sit' },
  { buttonIndex: STANDARD_BUTTONS.Y, modifier: 'RB', action: 'emote:fall' },
  { buttonIndex: STANDARD_BUTTONS.DPAD_UP, action: 'cycleCostume' },
  { buttonIndex: STANDARD_BUTTONS.DPAD_DOWN, action: 'die' },
  { buttonIndex: STANDARD_BUTTONS.L3, action: 'toggleRunLock' },
  { buttonIndex: STANDARD_BUTTONS.R3, action: 'resetFacing' },
  { buttonIndex: STANDARD_BUTTONS.START, action: 'toggleRecording' },
  { buttonIndex: STANDARD_BUTTONS.SELECT, action: 'toggleUI' },
];

export class CharacterController {
  private assignments: Map<number, number> = new Map(); // gamepadIndex -> slotIndex
  private characters: Map<number, Character> = new Map(); // slotIndex -> Character
  private bindings: ActionBinding[] = DEFAULT_BINDINGS;
  private runLock: Map<number, boolean> = new Map(); // slotIndex -> runLocked
  private walkThreshold = 0.4;

  constructor(private scene: Phaser.Scene) {
    eventBridge.on('gamepad:input', this.handleInput);
    eventBridge.on('character:assign-gamepad', ({ slotIndex, gamepadIndex }) => {
      this.assignGamepad(gamepadIndex, slotIndex);
    });
  }

  registerCharacter(character: Character): void {
    this.characters.set(character.slotIndex, character);
  }

  unregisterCharacter(slotIndex: number): void {
    this.characters.delete(slotIndex);
  }

  assignGamepad(gamepadIndex: number | null, slotIndex: number): void {
    // Remove any existing assignment for this gamepad
    if (gamepadIndex !== null) {
      for (const [gp, slot] of this.assignments) {
        if (gp === gamepadIndex) this.assignments.delete(gp);
      }
      this.assignments.set(gamepadIndex, slotIndex);
    }
  }

  private handleInput = ({ gamepadIndex, input }: {
    gamepadIndex: number;
    input: GamepadInput;
  }): void => {
    const slotIndex = this.assignments.get(gamepadIndex);
    if (slotIndex === undefined) return;
    const character = this.characters.get(slotIndex);
    if (!character) return;

    // --- Sticks ---
    const { leftStick, rightStick } = input;

    if (leftStick.magnitude > 0.01) {
      character.move(leftStick.x, leftStick.y, leftStick.magnitude);
    } else {
      character.stopMoving();
    }

    if (rightStick.magnitude > 0.1) {
      character.setLookDirection(this.angleToDirection(rightStick.angle));
    }

    // --- Buttons ---
    const lbHeld = input.buttons[STANDARD_BUTTONS.LB]?.held ?? false;
    const rbHeld = input.buttons[STANDARD_BUTTONS.RB]?.held ?? false;

    // Analog triggers (continuous hold)
    const ltValue = input.buttons[STANDARD_BUTTONS.LT]?.value ?? 0;
    const rtValue = input.buttons[STANDARD_BUTTONS.RT]?.value ?? 0;
    character.crouch(ltValue > 0.5);
    character.block(rtValue > 0.5);

    // Pressed-this-frame buttons
    for (const btn of input.buttons) {
      if (!btn.pressed) continue;
      const modifier = lbHeld ? 'LB' : rbHeld ? 'RB' : null;
      const binding = this.findBinding(btn.index, modifier);
      if (binding) this.executeAction(binding.action, character, slotIndex);
    }
  };

  private findBinding(buttonIndex: number, modifier: string | null): ActionBinding | null {
    // First try with modifier, then without
    return this.bindings.find(b =>
      b.buttonIndex === buttonIndex && b.modifier === modifier
    ) ?? (modifier ? this.bindings.find(b =>
      b.buttonIndex === buttonIndex && !b.modifier
    ) : null) ?? null;
  }

  private executeAction(action: string, character: Character, slotIndex: number): void {
    if (action.startsWith('emote:')) {
      character.emote(action.replace('emote:', ''));
    } else if (action === 'jump') {
      character.jump();
    } else if (action === 'attack1' || action === 'attack2' || action === 'action') {
      character.attack(action);
    } else if (action === 'die') {
      character.die();
    } else if (action === 'cycleCostume') {
      eventBridge.emit('character:set-costume', { slotIndex, costumeId: 'next' });
    } else if (action === 'toggleRunLock') {
      const current = this.runLock.get(slotIndex) ?? false;
      this.runLock.set(slotIndex, !current);
    } else if (action === 'resetFacing') {
      character.setLookDirection(character.direction);
    } else if (action === 'toggleRecording') {
      // Delegated to RecordingService via EventBridge
      eventBridge.emit('recording:countdown-start', undefined);
    } else if (action === 'toggleUI') {
      eventBridge.emit('ui:set-visibility', { visible: false }); // toggle handled by Vue
    }
  }

  private angleToDirection(angle: number): Direction {
    // Convert radians to 4-direction
    const deg = ((angle * 180 / Math.PI) + 360) % 360;
    if (deg >= 315 || deg < 45) return Direction.RIGHT;
    if (deg >= 45 && deg < 135) return Direction.DOWN;
    if (deg >= 135 && deg < 225) return Direction.LEFT;
    return Direction.UP;
  }

  destroy(): void {
    eventBridge.off('gamepad:input', this.handleInput);
  }
}
```

### CameraController System

- **Responsibility**: Manages the Phaser camera with three modes — Fixed (full stage visible), Follow (tracks a specific character with smooth lerp), and Free Pan (manual control via keyboard or gamepad D-pad). Handles zoom. Coordinates with recording pipeline to ensure camera state is consistent.
- **Technology**: Phaser 3.90.0 Camera system
- **Interfaces**: Consumes `camera:set-mode` and `camera:set-zoom` events, emits `camera:mode-changed`.

```typescript
// src/phaser/systems/CameraController.ts

export enum CameraMode {
  FIXED = 'fixed',
  FOLLOW = 'follow',
  FREE_PAN = 'free_pan',
}

export interface CameraControllerConfig {
  stageWidth: number;    // full stage width in pixels
  stageHeight: number;   // full stage height in pixels
  viewportWidth: number; // canvas width
  viewportHeight: number;
  defaultZoom: number;   // default 1
  minZoom: number;       // 0.5
  maxZoom: number;       // 3
  followLerp: number;    // 0.08 (smooth follow speed)
  panSpeed: number;      // pixels per second for free pan
}

export class CameraController {
  private mode: CameraMode = CameraMode.FIXED;
  private camera: Phaser.Cameras.Scene2D.Camera;
  private targetCharacter: Character | null = null;

  constructor(private scene: Phaser.Scene, private config: CameraControllerConfig) {
    this.camera = scene.cameras.main;
    this.camera.setBounds(0, 0, config.stageWidth, config.stageHeight);
    eventBridge.on('camera:set-mode', this.setMode);
    eventBridge.on('camera:set-zoom', this.setZoom);
  }

  setMode = ({ mode, targetSlot }: { mode: CameraMode; targetSlot?: number }): void => {
    this.mode = mode;
    this.camera.stopFollow();
    switch (mode) {
      case CameraMode.FIXED:
        this.camera.centerOn(this.config.stageWidth / 2, this.config.stageHeight / 2);
        this.camera.setZoom(this.fitZoom());
        break;
      case CameraMode.FOLLOW:
        if (targetSlot !== undefined) {
          // Character lookup via scene registry
          const character = this.scene.registry.get(`character:${targetSlot}`) as Character;
          if (character) {
            this.targetCharacter = character;
            this.camera.startFollow(character.sprite, true, this.config.followLerp, this.config.followLerp);
          }
        }
        break;
      case CameraMode.FREE_PAN:
        // Pan handled in update loop
        break;
    }
    eventBridge.emit('camera:mode-changed', { mode });
  };

  setZoom = ({ zoom }: { zoom: number }): void => {
    this.camera.setZoom(
      Phaser.Math.Clamp(zoom, this.config.minZoom, this.config.maxZoom)
    );
  };

  update(delta: number): void {
    if (this.mode === CameraMode.FREE_PAN) {
      // Read keyboard arrows or dedicated pan input for camera movement
      const cursors = this.scene.input.keyboard?.createCursorKeys();
      if (cursors) {
        const speed = this.config.panSpeed * (delta / 1000);
        if (cursors.left.isDown) this.camera.scrollX -= speed;
        if (cursors.right.isDown) this.camera.scrollX += speed;
        if (cursors.up.isDown) this.camera.scrollY -= speed;
        if (cursors.down.isDown) this.camera.scrollY += speed;
      }
    }
  }

  private fitZoom(): number {
    return Math.min(
      this.config.viewportWidth / this.config.stageWidth,
      this.config.viewportHeight / this.config.stageHeight
    );
  }

  destroy(): void {
    eventBridge.off('camera:set-mode', this.setMode);
    eventBridge.off('camera:set-zoom', this.setZoom);
  }
}
```

### RecordingService

- **Responsibility**: Orchestrates the full recording pipeline — codec detection, stream composition (canvas video + microphone audio), MediaRecorder lifecycle, Blob accumulation, preview generation, and download. Implements the codec fallback chain: MP4/H.264 -> WebM/VP9 -> WebM/VP8. Monitors tab visibility via Page Visibility API. Tracks memory usage and enforces duration limits.
- **Technology**: MediaRecorder API, canvas.captureStream(), Page Visibility API, Blob API
- **Interfaces**: EventBridge events for recording lifecycle, direct method API for Vue composable.

```typescript
// src/services/RecordingService.ts

export interface CodecOption {
  mimeType: string;
  label: string;
  extension: string;
}

export interface RecordingConfig {
  targetFps: number;            // default 30
  videoBitrate: number;         // default 2_500_000 (2.5 Mbps)
  audioBitrate: number;         // default 128_000 (128 kbps)
  maxDurationMs: number;        // default 600_000 (10 min)
  memoryLimitMB: number;        // default 450
  timesliceMs: number;          // default 1000 (flush every 1s)
}

export type RecordingStatus = 'idle' | 'countdown' | 'recording' | 'stopping' | 'preview';

export const CODEC_FALLBACK_CHAIN: CodecOption[] = [
  { mimeType: 'video/mp4; codecs="avc1.42E01E, opus"',  label: 'MP4 (H.264 + Opus)', extension: 'mp4'  },
  { mimeType: 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"', label: 'MP4 (H.264 + AAC)', extension: 'mp4' },
  { mimeType: 'video/webm; codecs="vp9, opus"',         label: 'WebM (VP9 + Opus)',   extension: 'webm' },
  { mimeType: 'video/webm; codecs="vp8, opus"',         label: 'WebM (VP8 + Opus)',   extension: 'webm' },
  { mimeType: 'video/webm; codecs="vp8, vorbis"',       label: 'WebM (VP8 + Vorbis)', extension: 'webm' },
];

export class RecordingService {
  private config: RecordingConfig;
  private status: RecordingStatus = 'idle';
  private recorder: MediaRecorder | null = null;
  private chunks: Blob[] = [];
  private selectedCodec: CodecOption | null = null;
  private startTime = 0;
  private estimatedSizeMB = 0;
  private visibilityHandler: (() => void) | null = null;
  private durationTimer: ReturnType<typeof setTimeout> | null = null;

  constructor(config?: Partial<RecordingConfig>) {
    this.config = {
      targetFps: 30,
      videoBitrate: 2_500_000,
      audioBitrate: 128_000,
      maxDurationMs: 600_000,
      memoryLimitMB: 450,
      timesliceMs: 1000,
      ...config,
    };
    this.detectBestCodec();
  }

  /** Detect best supported codec at initialization */
  detectBestCodec(): CodecOption | null {
    for (const option of CODEC_FALLBACK_CHAIN) {
      if (MediaRecorder.isTypeSupported(option.mimeType)) {
        this.selectedCodec = option;
        return option;
      }
    }
    this.selectedCodec = null;
    return null;
  }

  getSelectedCodec(): CodecOption | null {
    return this.selectedCodec;
  }

  getStatus(): RecordingStatus {
    return this.status;
  }

  /** Start recording from a canvas + optional audio stream */
  async startRecording(canvas: HTMLCanvasElement, audioStream?: MediaStream): Promise<void> {
    if (!this.selectedCodec) {
      eventBridge.emit('recording:error', { message: 'No supported recording codec found in this browser.' });
      return;
    }

    // Compose streams
    const videoStream = canvas.captureStream(this.config.targetFps);
    const combinedStream = new MediaStream();
    for (const track of videoStream.getVideoTracks()) {
      combinedStream.addTrack(track);
    }
    if (audioStream) {
      for (const track of audioStream.getAudioTracks()) {
        combinedStream.addTrack(track);
      }
    }

    // Create MediaRecorder
    this.recorder = new MediaRecorder(combinedStream, {
      mimeType: this.selectedCodec.mimeType,
      videoBitsPerSecond: this.config.videoBitrate,
      audioBitsPerSecond: this.config.audioBitrate,
    });

    this.chunks = [];
    this.estimatedSizeMB = 0;
    this.startTime = Date.now();

    this.recorder.ondataavailable = (e: BlobEvent) => {
      if (e.data.size > 0) {
        this.chunks.push(e.data);
        this.estimatedSizeMB += e.data.size / (1024 * 1024);

        if (this.estimatedSizeMB >= this.config.memoryLimitMB) {
          eventBridge.emit('recording:memory-warning', {
            usedMB: Math.round(this.estimatedSizeMB),
            limitMB: this.config.memoryLimitMB,
          });
          this.stopRecording();
        }
      }
    };

    this.recorder.onerror = () => {
      eventBridge.emit('recording:error', { message: 'MediaRecorder error during recording.' });
      this.stopRecording();
    };

    this.recorder.onstop = () => {
      this.status = 'preview';
    };

    // Start
    this.recorder.start(this.config.timesliceMs);
    this.status = 'recording';
    eventBridge.emit('recording:started', undefined);

    // Duration limit
    this.durationTimer = setTimeout(() => this.stopRecording(), this.config.maxDurationMs);

    // Tab visibility monitoring
    this.installVisibilityMonitor();
  }

  stopRecording(): void {
    if (this.recorder && this.recorder.state !== 'inactive') {
      this.recorder.stop();
    }
    if (this.durationTimer) {
      clearTimeout(this.durationTimer);
      this.durationTimer = null;
    }
    this.removeVisibilityMonitor();
    this.status = 'stopping';
    eventBridge.emit('recording:stopped', undefined);
  }

  /** Get recorded blob for preview / download */
  getRecordedBlob(): Blob | null {
    if (this.chunks.length === 0) return null;
    return new Blob(this.chunks, { type: this.selectedCodec!.mimeType });
  }

  /** Create an object URL for preview playback */
  getPreviewUrl(): string | null {
    const blob = this.getRecordedBlob();
    return blob ? URL.createObjectURL(blob) : null;
  }

  /** Trigger browser download */
  downloadRecording(filename?: string): void {
    const blob = this.getRecordedBlob();
    if (!blob || !this.selectedCodec) return;
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = filename ?? `virtual-stage-${Date.now()}.${this.selectedCodec.extension}`;
    a.click();
    URL.revokeObjectURL(url);
  }

  /** Discard recording and free memory */
  discardRecording(): void {
    this.chunks = [];
    this.estimatedSizeMB = 0;
    this.status = 'idle';
  }

  getElapsedMs(): number {
    if (this.status !== 'recording') return 0;
    return Date.now() - this.startTime;
  }

  /** Build metadata for the recording */
  getMetadata(): RecordingMetadata {
    return {
      codec: this.selectedCodec?.label ?? 'unknown',
      mimeType: this.selectedCodec?.mimeType ?? 'unknown',
      durationMs: Date.now() - this.startTime,
      estimatedSizeMB: Math.round(this.estimatedSizeMB * 100) / 100,
      fps: this.config.targetFps,
      videoBitrate: this.config.videoBitrate,
      timestamp: new Date().toISOString(),
    };
  }

  // --- Tab visibility monitoring ---

  private installVisibilityMonitor(): void {
    this.visibilityHandler = () => {
      if (document.hidden) {
        eventBridge.emit('recording:visibility-lost', undefined);
      } else {
        eventBridge.emit('recording:visibility-restored', undefined);
      }
    };
    document.addEventListener('visibilitychange', this.visibilityHandler);
  }

  private removeVisibilityMonitor(): void {
    if (this.visibilityHandler) {
      document.removeEventListener('visibilitychange', this.visibilityHandler);
      this.visibilityHandler = null;
    }
  }

  destroy(): void {
    this.stopRecording();
    this.discardRecording();
    this.removeVisibilityMonitor();
  }
}
```

### AudioService

- **Responsibility**: Manages microphone capture via getUserMedia, routes audio through a Web Audio API pipeline (MediaStreamSource -> GainNode -> AnalyserNode -> MediaStreamDestination), provides volume control and level monitoring, and outputs a clean MediaStream for the RecordingService.
- **Technology**: Web Audio API, getUserMedia
- **Interfaces**: Direct API for Vue composable, EventBridge events for level and permission status.

```typescript
// src/services/AudioService.ts

export interface AudioServiceConfig {
  fftSize: number;           // default 256 (for level monitoring)
  smoothingTimeConstant: number; // default 0.8
  levelUpdateIntervalMs: number; // default 50 (20 updates/sec)
}

export class AudioService {
  private config: AudioServiceConfig;
  private audioContext: AudioContext | null = null;
  private sourceNode: MediaStreamAudioSourceNode | null = null;
  private gainNode: GainNode | null = null;
  private analyserNode: AnalyserNode | null = null;
  private destinationNode: MediaStreamAudioDestinationNode | null = null;
  private rawStream: MediaStream | null = null;
  private levelInterval: ReturnType<typeof setInterval> | null = null;
  private permissionGranted = false;

  constructor(config?: Partial<AudioServiceConfig>) {
    this.config = {
      fftSize: 256,
      smoothingTimeConstant: 0.8,
      levelUpdateIntervalMs: 50,
      ...config,
    };
  }

  /** Request microphone access and build audio pipeline */
  async initialize(): Promise<boolean> {
    try {
      this.rawStream = await navigator.mediaDevices.getUserMedia({ audio: true });
      this.audioContext = new AudioContext();
      this.sourceNode = this.audioContext.createMediaStreamSource(this.rawStream);
      this.gainNode = this.audioContext.createGain();
      this.analyserNode = this.audioContext.createAnalyser();
      this.analyserNode.fftSize = this.config.fftSize;
      this.analyserNode.smoothingTimeConstant = this.config.smoothingTimeConstant;
      this.destinationNode = this.audioContext.createMediaStreamDestination();

      // Pipeline: mic -> gain -> analyser -> destination (for recording)
      this.sourceNode.connect(this.gainNode);
      this.gainNode.connect(this.analyserNode);
      this.analyserNode.connect(this.destinationNode);
      // NOTE: We do NOT connect to audioContext.destination — no speaker playback

      this.permissionGranted = true;
      this.startLevelMonitoring();
      return true;
    } catch (err) {
      this.permissionGranted = false;
      eventBridge.emit('audio:permission-denied', undefined);
      return false;
    }
  }

  /** Get the processed audio stream for RecordingService */
  getOutputStream(): MediaStream | null {
    return this.destinationNode?.stream ?? null;
  }

  /** Set gain (0.0 to 2.0, where 1.0 = unity) */
  setGain(value: number): void {
    if (this.gainNode) {
      this.gainNode.gain.value = Math.max(0, Math.min(2, value));
    }
  }

  getGain(): number {
    return this.gainNode?.gain.value ?? 1;
  }

  isPermissionGranted(): boolean {
    return this.permissionGranted;
  }

  private startLevelMonitoring(): void {
    if (!this.analyserNode) return;
    const dataArray = new Uint8Array(this.analyserNode.frequencyBinCount);

    this.levelInterval = setInterval(() => {
      if (!this.analyserNode) return;
      this.analyserNode.getByteTimeDomainData(dataArray);

      let peak = 0;
      let sumSquares = 0;
      for (let i = 0; i < dataArray.length; i++) {
        const normalized = (dataArray[i] - 128) / 128; // -1 to 1
        peak = Math.max(peak, Math.abs(normalized));
        sumSquares += normalized * normalized;
      }
      const rms = Math.sqrt(sumSquares / dataArray.length);

      eventBridge.emit('audio:level', { peak, rms });
    }, this.config.levelUpdateIntervalMs);
  }

  destroy(): void {
    if (this.levelInterval) clearInterval(this.levelInterval);
    this.rawStream?.getTracks().forEach(t => t.stop());
    this.audioContext?.close();
    this.sourceNode = null;
    this.gainNode = null;
    this.analyserNode = null;
    this.destinationNode = null;
    this.rawStream = null;
  }
}
```

### SceneStateService

- **Responsibility**: Serializes and deserializes the complete scene state (background, characters, props, camera) to/from JSON. Provides save/load to localStorage, file export/import, and (Phase 6) Firestore sync. Validates scene JSON on import.
- **Technology**: JSON, localStorage, File API, (future) Firestore
- **Interfaces**: Direct method API, EventBridge for scene state snapshots.

```typescript
// src/services/SceneStateService.ts

export interface SceneState {
  version: number;           // schema version for migration
  name: string;
  background: BackgroundState;
  characters: CharacterSnapshot[];
  props: PropSnapshot[];
  camera: CameraState;
  metadata: SceneMetadata;
}

export interface BackgroundState {
  backgroundId: string;
  width: number;
  height: number;
}

export interface CharacterSnapshot {
  characterId: string;
  slotIndex: number;
  x: number;
  y: number;
  direction: Direction;
  lookDirection: Direction;
  state: CharacterState;
  costumeId: string;
  gamepadIndex: number | null;
}

export interface PropSnapshot {
  propInstanceId: string;
  propId: string;
  x: number;
  y: number;
  scaleX: number;
  scaleY: number;
  flipX: boolean;
  layer: 'foreground' | 'background';
  zIndex: number;           // Y-based depth sort value
}

export interface CameraState {
  mode: CameraMode;
  followTargetSlot: number | null;
  scrollX: number;
  scrollY: number;
  zoom: number;
}

export interface SceneMetadata {
  createdAt: string;        // ISO 8601
  updatedAt: string;        // ISO 8601
  appVersion: string;
  assetManifestVersion: string;
}

export class SceneStateService {
  private static STORAGE_PREFIX = 'virtualstage:scene:';
  private static CURRENT_VERSION = 1;

  /** Collect current state from Phaser via EventBridge request/response */
  requestSnapshot(): void {
    // Phaser scene listens for this and responds with 'scene:state-snapshot'
    eventBridge.emit('scene:load-state', undefined as any); // request pattern
  }

  /** Save to localStorage */
  saveLocal(name: string, state: SceneState): void {
    state.name = name;
    state.metadata.updatedAt = new Date().toISOString();
    localStorage.setItem(
      SceneStateService.STORAGE_PREFIX + name,
      JSON.stringify(state)
    );
  }

  /** Load from localStorage */
  loadLocal(name: string): SceneState | null {
    const raw = localStorage.getItem(SceneStateService.STORAGE_PREFIX + name);
    if (!raw) return null;
    const state = JSON.parse(raw) as SceneState;
    return this.validate(state) ? state : null;
  }

  /** List saved scenes from localStorage */
  listLocalSaves(): string[] {
    const names: string[] = [];
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key?.startsWith(SceneStateService.STORAGE_PREFIX)) {
        names.push(key.replace(SceneStateService.STORAGE_PREFIX, ''));
      }
    }
    return names;
  }

  /** Export scene state as downloadable JSON file */
  exportToFile(state: SceneState): void {
    const json = JSON.stringify(state, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `${state.name || 'scene'}.virtualstage.json`;
    a.click();
    URL.revokeObjectURL(url);
  }

  /** Import scene state from a JSON file */
  async importFromFile(file: File): Promise<SceneState | null> {
    const text = await file.text();
    try {
      const state = JSON.parse(text) as SceneState;
      if (!this.validate(state)) return null;
      return state;
    } catch {
      return null;
    }
  }

  /** Validate scene state structure */
  private validate(state: unknown): state is SceneState {
    if (!state || typeof state !== 'object') return false;
    const s = state as Record<string, unknown>;
    return (
      typeof s.version === 'number' &&
      typeof s.name === 'string' &&
      typeof s.background === 'object' &&
      Array.isArray(s.characters) &&
      Array.isArray(s.props) &&
      typeof s.camera === 'object' &&
      typeof s.metadata === 'object'
    );
  }
}
```

### AssetService

- **Responsibility**: Manages the asset manifest, coordinates sprite sheet loading into Phaser's texture manager, validates asset files (format, dimensions), and provides the asset catalog for Vue UI panels. Supports LPC 64x64 sprite format and Aseprite JSON metadata natively.
- **Technology**: Phaser loader, JSON manifest, Aseprite JSON format
- **Interfaces**: Direct API for Vue composable + Phaser scene preload integration, EventBridge for load progress.

```typescript
// src/services/AssetService.ts

export interface AssetManifest {
  version: string;
  characters: CharacterAssetDef[];
  backgrounds: BackgroundAssetDef[];
  props: PropAssetDef[];
}

export interface CharacterAssetDef {
  id: string;
  name: string;
  spriteSheet: string;        // path to PNG
  animationJson: string;      // path to Aseprite JSON or custom JSON
  format: SpriteFormat;
  costumes?: CostumeDef[];
  license: LicenseInfo;
  thumbnail: string;
}

export interface SpriteFormat {
  frameWidth: number;         // default 64 (LPC standard)
  frameHeight: number;        // default 64
  directions: 4 | 8;         // 4 for LPC, 8 for some itch.io packs
  padding: number;            // pixels between frames, default 0
  layout: 'grid' | 'strip' | 'aseprite';
}

export interface CostumeDef {
  id: string;
  name: string;
  spriteSheet: string;
  animationJson: string;
}

export interface BackgroundAssetDef {
  id: string;
  name: string;
  image: string;              // path to PNG/JPG
  width: number;
  height: number;
  thumbnail: string;
}

export interface PropAssetDef {
  id: string;
  name: string;
  image: string;
  width: number;
  height: number;
  thumbnail: string;
  origin?: { x: number; y: number }; // anchor point, default { x: 0.5, y: 1 } (bottom center)
}

export interface LicenseInfo {
  type: 'CC0' | 'CC-BY-SA-3.0' | 'GPL-3.0' | 'custom';
  attribution?: string;
  url?: string;
}

export class AssetService {
  private manifest: AssetManifest | null = null;
  private loadedTextures: Set<string> = new Set();

  async loadManifest(url: string): Promise<AssetManifest> {
    const response = await fetch(url);
    this.manifest = await response.json() as AssetManifest;
    return this.manifest;
  }

  getManifest(): AssetManifest | null {
    return this.manifest;
  }

  getCharacter(id: string): CharacterAssetDef | undefined {
    return this.manifest?.characters.find(c => c.id === id);
  }

  getBackground(id: string): BackgroundAssetDef | undefined {
    return this.manifest?.backgrounds.find(b => b.id === id);
  }

  getProp(id: string): PropAssetDef | undefined {
    return this.manifest?.props.find(p => p.id === id);
  }

  /** Load character assets into Phaser's texture manager */
  loadCharacterIntoPhaser(scene: Phaser.Scene, charDef: CharacterAssetDef): void {
    if (this.loadedTextures.has(charDef.id)) return;

    if (charDef.format.layout === 'aseprite') {
      scene.load.aseprite(charDef.id, charDef.spriteSheet, charDef.animationJson);
    } else {
      scene.load.spritesheet(charDef.id, charDef.spriteSheet, {
        frameWidth: charDef.format.frameWidth,
        frameHeight: charDef.format.frameHeight,
        spacing: charDef.format.padding,
      });
    }

    scene.load.once('complete', () => {
      this.loadedTextures.add(charDef.id);
      eventBridge.emit('asset:load-complete', undefined);
    });
    scene.load.start();
  }

  /** Get all license attributions for loaded assets */
  getAttributions(): LicenseInfo[] {
    if (!this.manifest) return [];
    const all = [
      ...this.manifest.characters.map(c => c.license),
      // backgrounds and props may also have license info
    ];
    return all.filter(l => l.type !== 'CC0' && l.attribution);
  }
}
```

---

## Data Model

### Scene State (JSON Schema)

```json
{
  "version": 1,
  "name": "My Scene",
  "background": {
    "backgroundId": "castle-hall",
    "width": 1920,
    "height": 1080
  },
  "characters": [
    {
      "characterId": "knight",
      "slotIndex": 0,
      "x": 400,
      "y": 500,
      "direction": "down",
      "lookDirection": "right",
      "state": "IDLE",
      "costumeId": "knight-default",
      "gamepadIndex": 0
    }
  ],
  "props": [
    {
      "propInstanceId": "prop-uuid-1",
      "propId": "barrel",
      "x": 600,
      "y": 520,
      "scaleX": 1,
      "scaleY": 1,
      "flipX": false,
      "layer": "foreground",
      "zIndex": 520
    }
  ],
  "camera": {
    "mode": "fixed",
    "followTargetSlot": null,
    "scrollX": 0,
    "scrollY": 0,
    "zoom": 1
  },
  "metadata": {
    "createdAt": "2026-04-12T14:30:00Z",
    "updatedAt": "2026-04-12T15:00:00Z",
    "appVersion": "0.1.0",
    "assetManifestVersion": "1.0.0"
  }
}
```

### Asset Manifest

```json
{
  "version": "1.0.0",
  "characters": [
    {
      "id": "knight",
      "name": "Knight",
      "spriteSheet": "assets/characters/knight/knight.png",
      "animationJson": "assets/characters/knight/knight.json",
      "format": {
        "frameWidth": 64,
        "frameHeight": 64,
        "directions": 4,
        "padding": 0,
        "layout": "grid"
      },
      "costumes": [
        {
          "id": "knight-gold",
          "name": "Gold Knight",
          "spriteSheet": "assets/characters/knight/knight-gold.png",
          "animationJson": "assets/characters/knight/knight.json"
        }
      ],
      "license": {
        "type": "CC-BY-SA-3.0",
        "attribution": "Universal LPC Spritesheet Character Generator contributors",
        "url": "https://liberatedpixelcup.github.io"
      },
      "thumbnail": "assets/characters/knight/thumb.png"
    }
  ],
  "backgrounds": [
    {
      "id": "castle-hall",
      "name": "Castle Hall",
      "image": "assets/backgrounds/castle-hall.png",
      "width": 1920,
      "height": 1080,
      "thumbnail": "assets/backgrounds/castle-hall-thumb.png"
    }
  ],
  "props": [
    {
      "id": "barrel",
      "name": "Barrel",
      "image": "assets/props/barrel.png",
      "width": 32,
      "height": 48,
      "thumbnail": "assets/props/barrel-thumb.png"
    }
  ]
}
```

### Recording Metadata

```typescript
export interface RecordingMetadata {
  codec: string;
  mimeType: string;
  durationMs: number;
  estimatedSizeMB: number;
  fps: number;
  videoBitrate: number;
  timestamp: string;          // ISO 8601
  sceneName?: string;
  sceneState?: SceneState;    // optional scene snapshot at recording time
}
```

---

## Technology Stack

| Layer | Choice | Rationale | Alternatives Considered |
|-------|--------|-----------|------------------------|
| **UI Framework** | Vue 3 (Composition API) | Reactive UI overlays, composable pattern for service wrappers, strong TypeScript support. Pre-decided by project owner. | React (rejected: project owner preference), Svelte (smaller community for Phaser integration) |
| **2D Engine** | Phaser 3.90.0 | Final stable v3, decade of production use, mature plugin ecosystem (rex, merged-input), official Vue TS template, strong sprite animation and camera systems. | Phaser 4.0.0 (rejected: 2-day-old release, ecosystem not ready — see ADR-001), PixiJS v8 (lower-level, no built-in camera/physics/animation systems), Kaplay/Kaboom (smaller ecosystem) |
| **Language** | TypeScript (strict mode) | Compile-time safety for EventBridge types, service interfaces, and scene state schema. Essential for a dual-framework app where type boundaries prevent integration bugs. | JavaScript (rejected: type safety is critical for the Vue-Phaser boundary) |
| **Build Tool** | Vite 6.x | Native ESM, fast HMR for Vue, official Phaser template uses Vite. | Webpack (slower, more complex config), Turbopack (less mature) |
| **Vue-Phaser Bridge** | mitt 3.x (EventBridge) | Tiny (200 bytes), typed, framework-agnostic. Decouples Vue and Phaser lifecycles completely. | Pinia store (rejected: creates shared mutable state across frameworks — see ADR-003), Phavuer (rejected: couples to Phaser 3 object model, hinders v4 migration), Custom EventTarget (larger, no tree-shaking) |
| **State Management** | Pinia (Vue UI state only) | Vue panel visibility, selected tools, user preferences. Small surface. | Vuex (legacy), composables-only (feasible but Pinia provides devtools integration) |
| **Video Recording** | MediaRecorder API + canvas.captureStream() | Browser-native, zero bundle cost, direct MP4 on Chrome 126+. | ffmpeg.wasm for primary path (rejected: 22MB bundle, 5-24x slower — see ADR-004), WebCodecs API (too low-level, still experimental) |
| **Audio Capture** | Web Audio API + getUserMedia | Native browser APIs, GainNode for volume control, AnalyserNode for VU meter, MediaStreamDestination for recording pipeline integration. | Third-party audio libs (unnecessary abstraction over well-supported native APIs) |
| **Backend** | Firebase (Auth + Firestore + Cloud Storage + Hosting) | Zero-server architecture, generous free tier, real-time sync for future collaboration, CDN-backed hosting. Pre-decided by project owner. | Supabase (rejected: project owner preference), self-hosted (rejected: operational burden for a creative tool) |
| **Hosting** | Firebase Hosting | SPA hosting with CDN, custom domain, automatic SSL, deploy via CLI. | Vercel (good but Firebase already in stack), Netlify (same), Cloudflare Pages (same) |
| **Asset Format** | LPC 64x64 (default) + Aseprite JSON | LPC is the closest industry standard for multi-directional pixel art sprites with extensive free CC0/CC-BY-SA assets. Aseprite JSON is the de facto sprite sheet metadata format. | Custom format (rejected: no ecosystem, no existing assets), Spine JSON (overkill for frame-based animation) |
| **Testing** | Vitest (unit) + Playwright (e2e) | Vitest integrates natively with Vite. Playwright supports gamepad API mocking and canvas testing. | Jest (slower with ESM), Cypress (weaker canvas support) |

---

## Infrastructure

### Firebase Hosting

- **Deployment**: `firebase deploy --only hosting` via CI/CD (GitHub Actions)
- **SPA Configuration**: All routes rewrite to `index.html`
- **Cache Headers**: Immutable hashing for JS/CSS bundles (`vite build` output), short cache for `index.html`, long cache for static assets
- **CDN**: Firebase's global CDN serves the SPA from edge nodes
- **Custom Domain**: Configured via Firebase console
- **COOP/COEP Headers**: Required only if ffmpeg.wasm multi-thread is used (Firefox fallback). Configured via `firebase.json` headers section when needed.

### Firestore Schema (Phase 6)

```
users/
  {userId}/
    profile: { displayName, createdAt, plan }
    settings: { defaultDeadzone, preferredCodec, ... }

scenes/
  {sceneId}/
    ownerId: string
    name: string
    state: SceneState          (full JSON)
    isPublic: boolean
    createdAt: timestamp
    updatedAt: timestamp
    sharedWith: string[]       (user IDs for future collaboration)

recordings/
  {recordingId}/
    ownerId: string
    sceneId: string | null
    metadata: RecordingMetadata
    storageRef: string         (Cloud Storage path)
    createdAt: timestamp
    isPublic: boolean
```

### Cloud Storage Structure

```
recordings/
  {userId}/
    {recordingId}.mp4          (or .webm)
    {recordingId}-thumb.png    (auto-generated thumbnail)

assets/
  user-uploads/                (future: user-uploaded custom sprites)
    {userId}/
      {assetId}/
        sprite.png
        animation.json
```

### Security Rules Overview

```javascript
// Firestore Rules (simplified)
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    match /scenes/{sceneId} {
      allow read: if resource.data.isPublic == true
                  || request.auth.uid == resource.data.ownerId
                  || request.auth.uid in resource.data.sharedWith;
      allow create: if request.auth != null;
      allow update, delete: if request.auth.uid == resource.data.ownerId;
    }
    match /recordings/{recordingId} {
      allow read: if resource.data.isPublic == true
                  || request.auth.uid == resource.data.ownerId;
      allow create: if request.auth != null;
      allow delete: if request.auth.uid == resource.data.ownerId;
    }
  }
}
```

```javascript
// Cloud Storage Rules (simplified)
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /recordings/{userId}/{allPaths=**} {
      allow read: if true;  // public recordings served via signed URLs
      allow write: if request.auth.uid == userId
                   && request.resource.size < 500 * 1024 * 1024  // 500MB max
                   && request.resource.contentType.matches('video/.*');
    }
    match /assets/user-uploads/{userId}/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth.uid == userId
                   && request.resource.size < 10 * 1024 * 1024   // 10MB max per asset
                   && (request.resource.contentType.matches('image/png')
                       || request.resource.contentType == 'application/json');
    }
  }
}
```

### Monitoring and Observability

- **Firebase Performance Monitoring**: Automatic page load and network request tracking (free tier)
- **Error Tracking**: Vue `app.config.errorHandler` + Phaser `game.events.on('error')` forwarded to Firebase Crashlytics (or a lightweight alternative like Sentry free tier)
- **Recording Pipeline Metrics**: Custom events tracking codec selected, recording duration, success/failure rate, file size — logged to Firebase Analytics
- **Gamepad Metrics**: Track controller types connected, disconnection rates, input latency (for debugging hardware-specific issues)

---

## Technical Risks

### RISK-001: Background Tab Throttling Destroys Recordings

- **Impact**: **Critical**. If a user switches tabs during recording, Chrome reduces requestAnimationFrame to ~1fps, producing unwatchable choppy output. The user loses their entire take with no recovery path.
- **Probability**: High. Tab switching is a natural user behavior (checking chat, reading scripts, responding to notifications).
- **Mitigation**:
  1. Page Visibility API monitoring (`document.hidden`) with immediate user notification
  2. Option to auto-pause recording on tab switch (with a "paused" overlay on canvas)
  3. Prominent "Recording Active" banner that is visible even when UI is hidden
  4. Recommend fullscreen mode (`document.requestFullscreen()`) during recording, which prevents accidental tab switches
  5. Display a post-recording quality warning if any visibility loss was detected during the take
- **Residual Risk**: Users may still ignore warnings. This is an inherent browser limitation.

### RISK-002: MediaRecorder Codec Compatibility Across Browsers

- **Impact**: **High**. Firefox only supports VP8/WebM. Safari is not targeted but has H.264/MP4. The fallback chain must work correctly or users get no recording at all.
- **Probability**: Medium. Chrome/Edge (primary targets) have excellent support. Firefox is secondary.
- **Mitigation**:
  1. Runtime codec detection via `MediaRecorder.isTypeSupported()` at app initialization
  2. Display the detected codec to the user in recording controls
  3. Graceful fallback chain: MP4/H.264 -> WebM/VP9 -> WebM/VP8 -> clear error
  4. Browser detection with a "for best experience, use Chrome or Edge" banner on Firefox
  5. Lazy-load ffmpeg.wasm only for Firefox users who want MP4 conversion

### RISK-003: Phaser v3 End-of-Life

- **Impact**: **Medium**. Phaser 3.90.0 is the final v3 release. No further bug fixes or security patches. If a critical Phaser bug is discovered, there is no official fix path.
- **Probability**: Low-medium. v3 is extremely mature (decade of use, tens of thousands of games). Major undiscovered bugs are unlikely.
- **Mitigation**:
  1. EventBridge architecture minimizes Phaser coupling surface — most business logic lives in framework-agnostic services
  2. Plan v4 migration for Phase 5/6 (post-MVP) once the ecosystem stabilizes (estimated Q3-Q4 2026)
  3. Avoid deep dependencies on Phaser renderer internals (custom shaders, pipeline extensions)
  4. Pin Phaser version in package.json to prevent accidental v4 upgrade
  5. Fork Phaser 3.90.0 only if a critical bug surfaces (last resort)

### RISK-004: Memory Pressure During Long Recordings

- **Impact**: **High**. MediaRecorder accumulates Blob chunks in memory. A 10-minute recording at 2.5 Mbps produces ~190MB of data. Combined with Phaser textures and the DOM, this approaches browser memory limits, especially on 8GB machines.
- **Probability**: Medium. Primary use case (30s-5min clips) is safe. Tertiary users (indie filmmakers wanting 30-minute sessions) will hit limits.
- **Mitigation**:
  1. Configurable duration cap (default 10 minutes) with user warning at 80% mark
  2. Track Blob size accumulation in RecordingService and emit memory warnings
  3. Use `MediaRecorder.requestData()` with `timesliceMs` to prevent unbounded buffering
  4. Document memory requirements and recommend closing other tabs during recording
  5. Future: investigate streaming Blobs to IndexedDB or Cloud Storage to reduce in-memory pressure

### RISK-005: Gamepad API Inconsistencies Across Controllers

- **Impact**: **Medium**. DualSense touchpad, Switch Pro button mapping, trigger dead zones, and vibration support vary across browsers and OSes. Users may experience unexpected button behavior.
- **Probability**: Medium. Xbox controllers are reliable. Non-Xbox controllers have documented quirks.
- **Mitigation**:
  1. Standard Gamepad layout (W3C) as the only supported mapping — controllers that report `mapping: "standard"` work best
  2. Remappable bindings (Action Binding Editor) so users can fix any mapping issues
  3. Per-controller deadzone configuration
  4. Detect controller type from `gamepad.id` string and apply known quirk corrections
  5. Test matrix: Xbox (USB + BT), DualSense (USB + BT), Switch Pro (USB + BT) on Chrome, Edge, Firefox
  6. Display "connected controller" info in UI so users can verify detection

### RISK-006: Vue-Phaser Lifecycle Desynchronization

- **Impact**: **Medium**. If Phaser's scene is destroyed before Vue components that reference it (or vice versa), null pointer errors and memory leaks result. Hot Module Replacement (HMR) during development is particularly prone to this.
- **Probability**: Medium during development, low in production.
- **Mitigation**:
  1. PhaserCanvas.vue owns the Phaser.Game lifecycle exclusively — `onBeforeUnmount` calls `game.destroy(true)`
  2. All Phaser references in Vue use `shallowRef` + `markRaw` to avoid proxy wrapping
  3. EventBridge listeners are registered with cleanup in `onBeforeUnmount` / scene `shutdown` event
  4. Phaser game objects are never stored in Pinia or reactive Vue state
  5. Development-mode guard: check `game.isRunning` before emitting events from Vue

### RISK-007: Firefox Audio Track Memory Leak

- **Impact**: **Medium**. Adding an audio track to a canvas stream via `addTrack()` causes constantly increasing RAM usage in Firefox, potentially exhausting system RAM in extended sessions.
- **Probability**: High on Firefox. Confirmed browser bug (Bug 1557980).
- **Mitigation**:
  1. Document Firefox recording as a "known limitation" with a recommendation to use Chrome/Edge
  2. On Firefox, offer video-only recording as a safer option
  3. If Firefox recording is attempted with audio, enforce a stricter duration limit (5 minutes)
  4. Monitor memory usage and auto-stop recording if RAM consumption exceeds threshold

---

## Architecture Decision Records

### ADR-001: Phaser 3.90.0 over Phaser 4.0.0

- **Context**: Phaser 4.0.0 "Caladan" released April 10, 2026 — two days before this analysis. v4 offers significant rendering improvements (SpriteGPULayer for 1M sprites, unified Filter system with Pixelate/Glow/Bloom, Render Node Architecture). However, Virtual Stage needs a stable, proven ecosystem for a complex dual-framework app. The project's rendering demands (a handful of sprites, simple backgrounds) do not require v4's performance capabilities.
- **Decision**: Start with Phaser 3.90.0 (final v3 release). The official `phaserjs/template-vue-ts` (Vite + TypeScript) is the project scaffold. Rex plugins (100+ UI components), phaser3-merged-input, and all community Vue integration patterns target v3. The EventBridge architecture minimizes Phaser-version coupling, making future migration tractable.
- **Consequences**:
  - (+) Immediate access to mature ecosystem: Vue template, rex plugins, merged-input, thousands of examples
  - (+) No risk from day-two v4 bugs or undiscovered breaking changes
  - (+) Community support (Stack Overflow, Discord) overwhelmingly targets v3
  - (-) Building on an EOL framework — no future patches from Phaser Studio
  - (-) v4's Filter system (Pixelate, Bloom, Vignette, Wipe) would be valuable for scene transitions — must be built manually or deferred to v4 migration
  - (-) Migration to v4 will be required eventually (planned for Phase 5/6, Q3-Q4 2026)
  - **Migration path**: The EventBridge pattern, framework-agnostic services, and avoidance of Phavuer (which wraps v3 objects as Vue components) mean the Phaser-touching code is concentrated in `src/phaser/`. Migration involves updating scene code, not service code.

### ADR-002: Standalone GamepadService over Phaser GamepadPlugin

- **Context**: Phaser 3 includes a built-in GamepadPlugin that handles up to 4 gamepads with button/stick state tracking. However, Vue components also need gamepad state for the Binding Editor, character assignment UI, and recording controls (Start button). Phaser's plugin is only accessible within Phaser scene update loops.
- **Decision**: Implement GamepadService as a standalone TypeScript class in `src/services/` that polls `navigator.getGamepads()` via its own `requestAnimationFrame` loop. It emits normalized input events through the EventBridge, consumable by both Vue (via composable) and Phaser (via CharacterController).
- **Consequences**:
  - (+) Both frameworks can access gamepad state without coupling
  - (+) GamepadService survives Phaser scene transitions (scene restart, switch)
  - (+) Deadzone, debounce, and button state logic centralized in one place
  - (+) Easier to unit test (no Phaser dependency)
  - (+) If migrating to Phaser v4, GamepadService needs zero changes
  - (-) Runs its own rAF loop alongside Phaser's game loop — two polling loops reading `navigator.getGamepads()` per frame. Negligible performance impact (getGamepads is a snapshot read, not a DOM query), but architecturally impure.
  - (-) Cannot use Phaser's built-in gamepad vibration helpers — must implement vibration via the Gamepad Extensions API directly if needed.

### ADR-003: mitt EventBridge over Pinia for Vue-Phaser Communication

- **Context**: Three patterns exist for Vue-Phaser communication: (a) shared Pinia store that both frameworks read/write, (b) mitt event emitter with typed events, (c) Phavuer declarative wrapper. The project needs a clean boundary between Vue's reactive world and Phaser's imperative game loop.
- **Decision**: Use mitt (200-byte typed event emitter) as the sole communication channel between Vue and Phaser. Pinia is used only for Vue-internal UI state (panel visibility, selected tools). Phaser never reads from or writes to Pinia.
- **Consequences**:
  - (+) Zero coupling between framework lifecycles — either can be restarted independently
  - (+) All cross-framework data flows are explicit and auditable (search for `eventBridge.emit` / `eventBridge.on`)
  - (+) Type safety via `EventMap` — TypeScript catches payload mismatches at compile time
  - (+) Phaser version migration only affects Phaser-side event handlers, not Vue stores
  - (+) mitt is 200 bytes gzipped — essentially zero bundle cost
  - (-) Event-based communication is less discoverable than direct state reads — developers must consult the EventMap type to understand data flows
  - (-) No built-in time-travel debugging (Pinia has Vue Devtools integration). Can be mitigated with a development-mode event logger middleware.
  - (-) Request-response patterns are more awkward than with shared state (must use event pairs: emit request, listen for response)

### ADR-004: Direct MP4 Recording with WebM Fallback over WebM-Only + ffmpeg.wasm

- **Context**: The original project concept specified WebM output via MediaRecorder with optional ffmpeg.wasm conversion to MP4 for social media compatibility. Research revealed that Chrome 126+ (the primary target browser) supports direct MP4/H.264 recording via MediaRecorder, producing files immediately compatible with TikTok, YouTube, and Instagram.
- **Decision**: The primary recording path on Chrome/Edge uses `video/mp4; codecs="avc1.42E01E, opus"` via MediaRecorder, producing native MP4 files. The fallback chain is WebM/VP9 then WebM/VP8 for browsers that do not support MP4 recording. ffmpeg.wasm is not included in the initial bundle; it is lazy-loaded only for Firefox users who request MP4 conversion.
- **Consequences**:
  - (+) Zero post-processing for the majority of users (Chrome/Edge) — recording output is immediately shareable
  - (+) Eliminates the 22MB ffmpeg.wasm bundle from the critical path
  - (+) MP4 avoids the WebM duration metadata bug that causes seek issues in some players
  - (+) Dramatically simpler recording pipeline for the primary path
  - (-) Firefox users get WebM output, which is less compatible with social media platforms
  - (-) Must maintain runtime codec detection and a fallback chain — more conditional logic than a single-format approach
  - (-) Codec detection must be tested across browser versions to ensure the fallback chain is reliable
  - **Risk**: A future Chrome update could change MediaRecorder codec behavior. The `isTypeSupported()` check makes this self-healing.

### ADR-005: Local-First with Optional Firebase Cloud Layer

- **Context**: The development phases defer Firebase to Phase 6. The core use case (record and download a video) requires no server at all. However, the roadmap includes cloud save, recording storage, and sharing — all of which need a backend.
- **Decision**: Every feature works fully offline using browser-local storage (localStorage for scene state, Blob URLs for recordings, File API for import/export). Firebase services are additive: authentication unlocks cloud save/sync, Cloud Storage unlocks recording upload/sharing, but the app remains fully functional without them. The service layer uses an abstract persistence interface that localStorage-backed implementations fulfill by default, with Firestore-backed implementations injected when the user authenticates.
- **Consequences**:
  - (+) Zero server cost and zero deployment complexity for MVP
  - (+) Works offline and in restricted network environments
  - (+) Firebase features can be developed and shipped incrementally without blocking core functionality
  - (+) Users who never sign up still get the full tool — no artificial friction
  - (-) Dual storage backends increase testing surface (must verify both localStorage and Firestore paths)
  - (-) Conflict resolution needed if a user edits a scene locally and the cloud version diverges (last-write-wins is acceptable for V1)
  - (-) Cloud Storage upload requires managing large file uploads (recordings can be hundreds of MB) with resume support

### ADR-006: LPC 64x64 as Default Sprite Format with Configurable Frame Sizes

- **Context**: Pixel art sprite sheets have no universal standard, but the LPC (Liberated Pixel Cup) system is the closest industry convention: 64x64 pixel frames, 4-direction grid layout, extensive CC0/CC-BY-SA assets, and a browser-based character generator. Many itch.io asset packs use 32x32 or 16x16 frames. The asset system must support the LPC ecosystem while remaining flexible.
- **Decision**: LPC 64x64 is the default and recommended sprite format. The `SpriteFormat` type in the asset manifest allows per-character `frameWidth`, `frameHeight`, and `directions` configuration. The AssetService supports both LPC grid layout and Aseprite JSON layout natively. The animation system reads frame definitions from JSON metadata rather than hardcoding frame positions.
- **Consequences**:
  - (+) Immediate access to the LPC asset ecosystem — hundreds of characters, costumes, and animations available for free
  - (+) Aseprite JSON support means any artist using Aseprite (the dominant pixel art tool) can export assets directly
  - (+) Configurable frame sizes allow supporting 32x32 and 16x16 packs from itch.io without code changes
  - (+) Data-driven animation definitions (JSON) mean new characters are added by editing the manifest, not code
  - (-) Mixed frame sizes in a single scene may look inconsistent (a 64x64 knight next to a 16x16 prop). Must handle scale normalization in the rendering layer.
  - (-) LPC's GPL 3.0 / CC-BY-SA 3.0 licensing requires attribution display — the UI must include an attributions panel

---

## Cross-Cutting Concerns

### Performance Budget

| Metric | Target | Measurement |
|--------|--------|-------------|
| Initial page load (10 Mbps) | < 5 seconds | Lighthouse, WebPageTest |
| Phaser game loop | 60 FPS stable | Phaser debug overlay, `performance.now()` |
| Recording capture | 30 FPS without dropping game loop below 50 FPS | Custom frame counter |
| Input-to-visual latency | < 50ms (3 frames at 60 FPS) | Manual measurement with high-speed camera or `performance.now()` delta |
| JS bundle size (gzipped) | < 300KB (excluding Phaser) | Vite build analysis |
| Phaser library size | ~1MB gzipped (v3.90.0) | Fixed cost, loaded once |
| Memory during 10-min recording | < 500MB total | Chrome DevTools Memory panel |

### Error Handling Strategy

1. **Service errors**: Each service catches its own errors and emits them via EventBridge (`recording:error`, `asset:load-error`). Vue components display toast notifications.
2. **Phaser errors**: `game.events.on('error')` catches unhandled Phaser errors and forwards to Vue via EventBridge.
3. **Gamepad disconnection**: Non-blocking notification; character freezes; recording continues.
4. **MediaRecorder failure**: Attempt to finalize partial recording; present whatever was captured.
5. **Asset loading failure**: Show placeholder sprite/background; log error; allow user to retry.

### Accessibility

- Vue UI components follow ARIA guidelines (role, aria-label, keyboard navigation)
- "No gamepad detected" state provides clear guidance
- Keyboard fallback (REQ-F084) provides basic character control: WASD movement, arrow keys for facing, number keys for actions
- High-contrast recording indicator (red dot + timer) visible against any background
- Audio VU meter provides visual feedback for hearing-impaired users monitoring mic input

---

*End of Architecture Analysis*
