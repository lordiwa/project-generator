# Intake — Virtual Stage

## Classification
- **Project Type**: software
- **Output Scale**: full (browser application with architecture, security, full plan)
- **Domain**: Creative Tools + Gaming Technology + Content Creation + Pixel Art
- **Geography**: Global (English-first, browser-based)
- **Language**: English

## Source Documents
1. `projectMaterial/virtual-stage/project-context.md` — Comprehensive project context covering vision, requirements, tech stack decision, gamepad mapping, recording pipeline design, architecture, development phases, competitive landscape, and risk assessment

## Project Summary

### Vision
A browser-based creative tool that turns gamepads into puppet controllers for pixel art characters. Users control characters in a 2D pixel art environment, speak dialogue through microphones, and record the resulting scenes as downloadable video files. A virtual puppet theater for pixel art machinima.

### Problem
Content creators who want to produce pixel-art animated scenes with voice acting have no accessible tool. Existing options require 3D engine expertise (Source Filmmaker), frame-by-frame animation skills (Aseprite), or programming knowledge (Unity/Godot). No tool lets someone pick up a gamepad, control a pixel art character in real-time, speak dialogue, and hit record.

### Core Concept
- **Gamepad as puppet controller**: Left joystick moves character, right joystick controls facing direction (look-at), face buttons trigger theatrical actions and emotes
- **Voice acting**: Microphone captures dialogue in real-time while user controls character
- **Video recording**: Canvas + audio stream recorded as WebM video via MediaRecorder API
- **Theatrical actions**: Attack, emotes, death — all acted, no game mechanics
- **Multi-puppet stage**: Up to 4 characters controlled by 4 gamepads simultaneously

### Target Users
| User Type | Use Case | Priority |
|-----------|----------|----------|
| Content Creators | Short animated skits (30s-5min) for YouTube/TikTok | Primary |
| Streamers | Real-time pixel art puppets during live streams | Secondary |
| Indie Filmmakers | Longer-form animated shorts (5-30min) | Tertiary |

### Tech Stack (Pre-decided)
| Layer | Technology |
|-------|------------|
| UI Framework | Vue 3 (Composition API + TypeScript) |
| 2D Engine | Phaser.js v3 |
| Build Tool | Vite |
| Vue-Phaser Bridge | mitt (EventEmitter) |
| Video Recording | MediaRecorder API (browser-native) |
| Audio Capture | Web Audio API + getUserMedia |
| Backend | Firebase (Auth + Firestore + Cloud Storage) |

### Key Technical Features
1. **Gamepad API integration** with dual-stick control (movement + look-at)
2. **Emote wheel system** (bumper modifiers + face buttons = 8 emotes)
3. **Character state machine** (IDLE, WALK, RUN, JUMP, CROUCH, ACTION, EMOTE, DEAD)
4. **captureStream + MediaRecorder pipeline** for WebM output
5. **Vue-Phaser boundary** via EventBridge (mitt) — strict separation of concerns
6. **Standalone GamepadService** accessible from both Vue and Phaser
7. **Scene serialization** to JSON for save/load
8. **Multi-gamepad support** (up to 4 simultaneous controllers)

### Art Assets Strategy
- Initial development: Open-license pixel art from itch.io
- MVP needs: 1 character with full animation set, 1 background
- Full set: 3 character types, 5 backgrounds, 15 props
- Future: User-uploaded custom assets

### Platform Target
- Browser-only (Chrome, Edge primary; Firefox secondary)
- No Electron, no mobile
- Safari not targeted (poor MediaRecorder + Gamepad API support)

### Development Phases
1. **Foundation** — Phaser + Vue shell, one character with gamepad movement
2. **Actions + Audio** — Full action system, emote wheel, mic capture
3. **Recording Pipeline (MVP)** — Record + preview + download WebM
4. **Multi-Character + Scenes** — Multiple gamepads, background selector, camera modes, save/load
5. **Props + Polish** — Prop placement, speech bubbles, transitions, onboarding
6. **Firebase + Cloud** — Auth, cloud save, recording storage, sharing

### Business Model
- Initially a free creative tool
- Future monetization: premium asset packs, cloud storage tiers, custom character commissions
- [HUMAN-INPUT-NEEDED] Monetization strategy not yet defined

### Competitive Landscape
No direct competitor exists. Adjacent tools:
- Source Filmmaker / Garry's Mod (3D, complex)
- VTuber software (face-tracking, not pixel art)
- Puppet Pals / Toontastic (mobile, not pixel art, no gamepad)
- RPG Maker (game creation, not scene acting)
- Aseprite (pixel art tool, not real-time)

## Domain Keywords
pixel art, gamepad, controller, machinima, puppeteering, scene acting, content creation, Phaser.js, Vue 3, MediaRecorder, captureStream, WebM, sprite animation, 2D game engine, creative tool, video recording, voice acting, emotes, character controller

## Expert Analysis Routing
| Agent | Runs | Priority Areas |
|-------|------|----------------|
| Research Analyst | Yes | 2D game engine ecosystem 2026, browser gamepad tools, machinima tools landscape, pixel art content creation market, MediaRecorder API state |
| Business Analyst | Yes | Use cases for all 3 user types, feature requirements formalization, workflow mapping, acceptance criteria |
| Product/UX | Yes | Character control UX, recording workflow, gamepad button mapping ergonomics, scene management UI, onboarding flow |
| Architect | Yes | Vue 3 + Phaser.js integration, EventBridge design, recording pipeline, GamepadService architecture, scene state serialization, Firebase schema |
| Security/Compliance | Yes | Microphone permissions/privacy, video storage security, user-uploaded assets validation, Firebase security rules |
| Project Manager | Yes | Phase planning, effort estimation, risk assessment, milestone definition, team requirements |
