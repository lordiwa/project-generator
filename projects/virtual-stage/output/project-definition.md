# Project Definition: Virtual Stage

## 1. Project Brief

**Vision**: A browser-based creative tool that turns gamepads into puppet controllers for pixel art characters. Users control characters on a 2D stage, speak dialogue through microphones, and record scenes as downloadable video files. A virtual puppet theater for pixel art machinima.

**Problem**: Content creators who want to produce pixel-art animated scenes with voice acting have no accessible tool. Existing options require 3D engine expertise (Source Filmmaker, iClone), frame-by-frame animation skills (Aseprite), or programming knowledge (Unity, Godot). No tool lets someone pick up a gamepad, control a pixel art character in real-time, speak dialogue, and hit record.

**Scope**:
- **In Scope**: Gamepad-controlled character movement and actions, microphone audio capture, video recording with MP4/WebM output, multi-character scenes (up to 4 gamepads), scene composition (backgrounds, props, camera), scene save/load, optional Firebase cloud layer
- **Out of Scope**: Game mechanics (hitboxes, damage, physics), mobile/tablet support, Safari support, 3D rendering, frame-by-frame animation editor, custom asset creation tools, live streaming integration (OBS captures the canvas directly)
- **Project Type**: Software | **Scale**: Full

### Key Findings

1. **The niche is genuinely unoccupied** -- After exhaustive analysis of machinima tools, VTuber tools, pixel art tools, and browser-based creative tools, no product exists that combines gamepad-controlled pixel art characters, a virtual stage, and integrated video recording. This is a first-mover play, not an incremental improvement.

2. **Chrome 126+ can record directly to MP4/H.264** -- This eliminates the need for ffmpeg.wasm conversion for the primary target browser. MP4 is the preferred format for TikTok, YouTube Shorts, and Instagram Reels. The recording pipeline is simpler than initially expected.

3. **Phaser 4.0.0 released April 10, 2026, but v3.90.0 is the correct starting point** -- The Vue templates, rex plugins, merged-input plugin, and community resources all target v3. The EventBridge architecture minimizes Phaser-version coupling for a future v4 migration.

4. **veadotube mini validates demand but leaves the performance gap open** -- The dominant PNGTuber app has gamepad support for expression switching, but no character movement on a stage. Its upcoming full version focuses on node-based animation, not gamepad puppeteering.

5. **The LPC sprite sheet system provides a ready-made asset foundation** -- 64x64 frames, 4-direction grid layout, extensive CC0/CC-BY-SA assets, and a browser-based character generator. Combined with itch.io packs, Virtual Stage can launch without commissioning custom art.

### Top Recommendations

1. **Validate the recording pipeline in Phase 1 as a proof-of-concept spike** -- captureStream + MediaRecorder is the highest-risk technical component. A 1-2 day PoC in Phase 1 validates the core assumption before investing in the full character system.

2. **Record directly to MP4/H.264 on Chrome/Edge** -- Use the codec fallback chain (MP4/H.264 → WebM/VP9 → WebM/VP8) with runtime `isTypeSupported()` detection. Reserve ffmpeg.wasm as a lazy-loaded fallback for Firefox only.

3. **Ship MVP (Phase 3) to 5-10 content creators before building Phase 4** -- Real user feedback on the gamepad-to-recording loop will surface UX issues no expert analysis can predict. Multi-character and cloud features should be informed by real usage.

4. **Position multi-gamepad collaborative recording as the viral growth mechanism** -- Four friends, four gamepads, four characters, one screen, one recording. This "pixel art improv theater" use case has no equivalent in any existing tool.

5. **Implement aggressive tab-visibility protection for recordings** -- Background tab throttling will silently destroy recordings. Page Visibility API auto-pause, fullscreen recommendation, and no-silent-corruption guarantees are essential.

---

## 2. Business Model

| Revenue Source | Description | Phase |
|----------------|-------------|-------|
| Free Tool | Core tool is free with no login required | 1-3 (MVP) |
| Cloud Storage Tiers | Free tier (limited saves) + paid tier for more storage | 6+ |
| Premium Asset Packs | Curated, high-quality character/background packs | Future |
| Custom Commissions | Character sprite creation service | Future |

**Core principle**: The tool works fully offline with no sign-in required. Firebase is an optional cloud layer for convenience. Revenue is deferred — validate the product first, monetize later.

[HUMAN-INPUT-NEEDED] Monetization strategy not yet defined. Does not block MVP.

---

## 3. Top Use Cases

**UC-1: Record a Solo Scene**. A content creator opens Virtual Stage in Chrome, connects an Xbox controller, selects a background and character, and presses Start to begin recording. They control the character with the left stick (walk/run), face the character with the right stick, trigger attacks with X/Y, emotes with LB/RB combos, and speak dialogue through their microphone. After 60 seconds, they press Start again to stop. A preview modal shows the clip. They click Download and receive an MP4 file ready for TikTok upload. Total time from opening the tool to downloaded video: under 5 minutes.

**UC-2: Record a Multi-Character Scene**. Four friends gather with four gamepads around one laptop connected to a TV. The host sets up a scene (castle background, 4 characters), assigns each gamepad to a character via the "press to identify" flow, positions a shared microphone centrally, and starts recording. All four puppeteer their characters simultaneously while improvising dialogue. After 3 minutes of pixel art improv theater, they stop recording, preview the chaos, and download the video to share in their group chat. Setup to recording: under 3 minutes.

**UC-3: Set Up and Save a Scene**. An indie filmmaker opens Virtual Stage, selects a forest background, places a knight and mage at opposite ends of the stage, adds prop barrels and torches for set dressing, sets the camera to Follow mode with 1.5x zoom on the knight, and saves the scene as "Chapter 3 - The Confrontation." They close the browser. Next day, they reload the scene from the save list, all positions and settings restored exactly, and begin recording multiple takes.

**UC-4: Live Stream with Pixel Art Puppet**. A Twitch streamer adds Virtual Stage as a browser source in OBS. They press Select on their gamepad to hide the Vue UI, leaving only the clean Phaser canvas. During their stream, they control a pixel art character with one gamepad while playing a game with keyboard/mouse on another monitor. The character walks, emotes, and reacts in real-time as the streamer talks to their audience. OBS captures the canvas directly — no in-app recording needed.

**UC-5: First-Time Experience**. A new user opens the URL with no prior knowledge. An interactive onboarding overlay detects their gamepad, walks them through moving (left stick), facing (right stick), acting (face buttons), and recording (Start button) in under 2 minutes. The onboarding ends with the user downloading their first 10-second clip — a tangible artifact from their first session.

---

## 4. Architecture Summary

### Tech Stack

| Layer | Technology | Rationale |
|-------|------------|-----------|
| UI Framework | Vue 3 (Composition API + TypeScript) | Preferred framework. Handles all DOM overlay UI |
| 2D Game Engine | Phaser.js 3.90.0 ("Tsugumi") | Final v3 release. Built-in animation manager, camera system, sprite handling. Mature ecosystem |
| Build Tool | Vite | Default for Vue 3, fast HMR, excellent Phaser compatibility |
| Vue-Phaser Bridge | mitt (200-byte typed EventEmitter) | Framework-agnostic. Neither Vue nor Phaser reaches into the other's state |
| Video Recording | MediaRecorder API (browser-native) | Direct MP4/H.264 on Chrome/Edge. WebM fallback on Firefox |
| Audio Capture | Web Audio API + getUserMedia | GainNode for volume control, AnalyserNode for VU meter |
| Backend (optional) | Firebase (Auth + Firestore + Cloud Storage + Hosting) | Cloud save for scenes/recordings. Added in Phase 6 |
| Language | TypeScript (strict mode) | Type safety across the Vue/Phaser/Services boundary |
| Validation | Zod | Scene JSON schema validation against prototype pollution and malicious payloads |

### System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│  BROWSER TAB                                                     │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  VUE 3 SHELL                                               │  │
│  │  TopBar | CharacterPanel | ScenePanel | RecordingControls  │  │
│  │  AudioControls | BindingEditor | PreviewModal | Onboarding │  │
│  └────────────────────────┬───────────────────────────────────┘  │
│                           │                                      │
│                   ┌───────▼───────┐                              │
│                   │  EVENT BRIDGE │  (mitt — typed events)        │
│                   └───────┬───────┘                              │
│                           │                                      │
│  ┌────────────────────────▼───────────────────────────────────┐  │
│  │  PHASER 3.90.0 CANVAS (WebGL)                              │  │
│  │  StageScene: Background + Characters (×4) + Props + Camera │  │
│  │  Systems: CharacterController + CameraController            │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  SHARED SERVICES (framework-agnostic TypeScript)            │  │
│  │  GamepadService | RecordingService | AudioService           │  │
│  │  SceneStateService | AssetService                           │  │
│  └────────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │  FIREBASE (optional — Phase 6)                              │  │
│  │  Auth | Firestore (scenes) | Cloud Storage (recordings)     │  │
│  └────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Key Architecture Decisions

| ADR | Decision | Rationale |
|-----|----------|-----------|
| ADR-001 | Phaser 3.90.0 over Phaser 4.0.0 | v4 released 2 days ago (April 10 2026). Vue templates, rex plugins, and Phavuer still target v3. EventBridge pattern minimizes migration surface for future v4 upgrade |
| ADR-002 | Standalone GamepadService over Phaser's GamepadPlugin | Both Vue components (binding editor, recording controls) and Phaser scenes need gamepad state. Standalone service avoids coupling to Phaser lifecycle |
| ADR-003 | mitt EventBridge over Pinia for Vue-Phaser communication | Framework-agnostic. Pinia is Vue-specific. mitt can be imported by both Vue and Phaser without coupling |
| ADR-004 | Direct MP4 recording with WebM fallback | Chrome 126+ supports `video/mp4; codecs="avc1.42E01E"` in MediaRecorder. Eliminates ffmpeg.wasm for primary browser. Fallback chain: MP4/H.264 → WebM/VP9 → WebM/VP8 |
| ADR-005 | Local-first, cloud-optional | All features work without sign-in. Firebase is additive. Reduces friction for new users. Two save/load paths (local + cloud) with identical scene JSON format |
| ADR-006 | LPC 64x64 as default sprite format | Closest thing to a sprite sheet standard. Extensive free assets. Aseprite JSON metadata support. Configurable frame sizes (32x32, 64x64) for non-LPC assets |

---

## 5. Security & Compliance Summary

**Risk profile**: Low-to-moderate. Free creative tool with no payment data, no health data, and no PII beyond optional Google auth profile. Real concerns are: microphone privacy, Firebase misconfiguration, and dependency supply chain.

**Must-do before cloud launch (Phase 6):**

- [ ] Fix 4 security issues found in architect's Firebase rules (forged ownerId on create, public recording reads, mutable ownership on update, no string length limits)
- [ ] Write Firebase security rules test suite BEFORE deploying rules (test-driven security)
- [ ] Configure Firebase App Check (reCAPTCHA Enterprise) to prevent API abuse
- [ ] Deploy Content Security Policy headers via firebase.json
- [ ] Publish Privacy Policy (GDPR: data collected, processing basis, retention, deletion rights)
- [ ] Publish Terms of Service (content policy, abuse reporting, DMCA process)
- [ ] Implement account deletion flow (user data + Firestore docs + Storage files)
- [ ] Enforce per-user Cloud Storage quota (2GB limit)
- [ ] Use signed URLs for recording sharing (not raw storage paths)

**Must-do for MVP (Phase 2-3):**

- [ ] Never auto-initialize microphone — require explicit user action (SEC-001)
- [ ] Show persistent visual indicator when mic is active (SEC-002)
- [ ] Stop all media tracks on page unload via `beforeunload` handler (SEC-003)
- [ ] Deploy CSP headers from Phase 3 onward (SEC-006)
- [ ] Enable `eslint-plugin-security` and ban `v-html` (SEC-008, SEC-009)
- [ ] Validate imported scene JSON with Zod schema against prototype pollution (SEC-007)

---

## 6. Gamepad Controls

```
Left Stick  = Movement (magnitude → walk/run speed)
Right Stick = Look-at direction (independent of movement)
A / Cross   = Jump
B / Circle  = Context action / Nod
X / Square  = Primary attack (theatrical)
Y / Triangle = Magic / Ranged (theatrical)
LB + A/B/X/Y = Emote wheel 1 (Wave, Shrug, Laugh, Cry)
RB + A/B/X/Y = Emote wheel 2 (Celebrate, Point, Sit, Fall)
LT (hold)   = Crouch
RT (hold)   = Shield Block
D-pad Up    = Cycle costume
D-pad Down  = Theatrical death
L3          = Toggle run lock
R3          = Reset facing to movement direction
Select      = Toggle UI visibility (clean recording)
Start       = Start/Stop recording
```

**Deadzone**: Radial with rescaling, default 0.15, user-configurable 0.00-0.25.

**State machine**: IDLE → WALK → RUN (joystick magnitude). JUMP, CROUCH, ACTION, EMOTE, DEAD triggered by buttons. Look-at direction processed in ALL states.

---

## 7. Roadmap

| Phase | Name | Duration | Key Deliverables | Milestone |
|-------|------|----------|-----------------|-----------|
| 1 | Foundation | 2.5-3.5 wk | Vite+Vue+Phaser scaffold, EventBridge, GamepadService, Character entity (state machine, 4-dir sprites), StageScene, one character + one background, recording PoC spike | **M1: Character Alive** — gamepad controls character on stage |
| 2 | Actions + Audio | 2-3 wk | Emote wheel (LB/RB combos), full 8-state machine, AudioService (getUserMedia + GainNode + VU meter), keyboard fallback, privacy notice before mic | **M2: Full Expression** — all actions working, mic capturing |
| 3 | Recording Pipeline | 3-4 wk | RecordingService (codec fallback chain), countdown, preview modal, MP4/WebM download, tab visibility auto-pause, CSP headers, ESLint security rules | **M3: MVP** — record + preview + download end-to-end |
| 4 | Multi-Character + Scenes | 4-6 wk | 4 simultaneous gamepads, character assignment UI, 3 char types + 5 backgrounds + 15 props, camera modes, scene save/load (local + file export), Zod validation, attribution panel | **M4: Party Mode** — 4-player collaborative recording |
| 5 | Polish + UX | 3-5 wk | Interactive onboarding (<2min), Quick Start mode, binding editor, settings, accessibility pass (WCAG AA), performance optimization | **M5: First-Time Ready** — approachable for new users |
| 6 | Firebase + Cloud | 4-6 wk | Firebase Auth, corrected Firestore/Storage rules + test suite, cloud save, recording upload, sharing links, App Check, privacy policy, ToS, account deletion | **M6: Cloud Launch** — optional cloud layer live |

**Timeline to MVP (M3)**: 7.5-10.5 weeks (~2-2.5 months)
**Timeline to full scope (M6)**: 19-27.5 weeks (~5-7 months)

```
Week:  1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26
       |---Phase 1---|---Phase 2---|-----Phase 3------|--------Phase 4---------|----Phase 5----|----Phase 6----|
                   M1           M2              M3                          M4              M5              M6
                                                 ^
                                           MVP GATE
```

**Critical path**: Phases 1→2→3 are strictly sequential (each depends on the prior). MVP at Phase 3. Ship to 5-10 content creators for feedback before Phase 4.

### If Behind Schedule

1. **Cut first**: Phase 6 (Firebase/Cloud) — tool works locally
2. **Cut second**: Phase 5 nice-to-haves (speech bubbles, transitions)
3. **Simplify**: Phase 4 props (fixed size, no drag/resize)
4. **Do not cut**: Phase 3 recording quality or Phase 2 keyboard fallback

### If Ahead of Schedule

1. **Add first**: MP4 quality presets (Low/Medium/High bitrate)
2. **Add second**: Chroma key / transparent background for OBS streamers
3. **Add third**: Aseprite JSON direct import for custom characters
4. **Add fourth**: Director mode (take numbering, scene markers)

---

## 8. Top 10 Risks

| # | Risk | Impact | Prob. | Mitigation |
|---|------|--------|-------|------------|
| 1 | Background tab throttling corrupts recordings | High | High | Page Visibility API auto-pause, fullscreen suggestion, no silent corruption |
| 2 | Vue-Phaser lifecycle conflicts (memory leaks, GC thrash) | High | Medium | `shallowRef`/`markRaw` for all Phaser refs. Never make Phaser objects reactive. Test early |
| 3 | Codec fragmentation across browsers | Medium | Medium | Runtime `isTypeSupported()` fallback chain. Accept Firefox is WebM-only |
| 4 | Sprite sheet format inconsistency across asset packs | Medium | High | Standardize on 64x64 LPC. AssetService handles configurable frame sizes. Budget extra time |
| 5 | Memory pressure during long recordings (>5 min) | High | Medium | 450MB blob limit, 10-min duration cap, memory warning events, auto-stop |
| 6 | Gamepad API inconsistency across controllers | Medium | Medium | Default to Standard Gamepad layout. Binding editor in Phase 5. Test with Xbox + DualSense |
| 7 | Firebase security misconfiguration (4 identified issues) | High | Low | Corrected rules provided by security analysis. Test-driven rules deployment. Block on test suite |
| 8 | Phaser v3 EOL — no upstream security fixes | Medium | Low | EventBridge minimizes coupling. Plan v4 migration post-Phase 6 once ecosystem stabilizes |
| 9 | Scope creep via nice-to-have features | Medium | High | Strict phase gates. MVP is Phase 3. P2 features wait. Use UX priority matrix as arbiter |
| 10 | veadotube full version adds spatial movement | Low | Low | Current roadmap focuses on node animation, not gamepad puppeteering. Monitor quarterly |

---

## 9. Open Questions

Decisions needed from the project owner:

1. **Maximum recording duration**: 10 min default is conservative. Indie filmmakers may want 15-20 min. Recommend: 10 min default, configurable up to 15 min.
2. **Keyboard fallback scope**: Full parity with gamepad or basic subset (WASD movement + face button actions + recording toggle)? Recommend: subset.
3. **Chroma key / transparent background**: High value for streamers (OBS integration). Add to Phase 5? Moderate effort.
4. **Asset licensing legal review**: GPL on sprite sheets used in a free tool is likely fine, but formal review is prudent before expanding the library.
5. **Monetization strategy**: Free tool forever? Freemium with cloud tiers? Premium asset marketplace? Does not block MVP.
6. **Multi-microphone support**: Current design uses one shared mic. Should individual per-gamepad mics be supported? Adds significant complexity.
7. **OBS integration**: Should Virtual Stage provide an NDI/virtual camera output, or is "capture the browser window in OBS" sufficient?

---

## 10. Suggested AI Agents

### Agent 1: Asset Curator — Pixel Art Asset Scout & Validator
**Description**: Searches itch.io and open-license repositories for pixel art sprite sheets matching Virtual Stage's format requirements. Validates frame sizes, animation coverage, licensing, and format compatibility.
**What it does**: Scans itch.io CC0/CC-BY-SA pixel art packs; validates sprite sheets against LPC format (64x64, 4-direction, JSON metadata); checks animation coverage (all 8 states + emotes); tracks attribution requirements; generates an asset manifest entry for each validated pack; flags format inconsistencies requiring adaptation.
**Tools needed**: WebSearch, WebFetch, Read, image analysis for sprite sheet validation.

### Agent 2: Gamepad Compatibility Tester — Cross-Controller QA Agent
**Description**: Generates and maintains a compatibility matrix of gamepad controllers across browsers. Documents button mapping quirks, deadzone behaviors, and known issues for Xbox, DualSense, Switch Pro, and third-party controllers.
**What it does**: Researches current Gamepad API browser support; documents per-controller mapping (axes, buttons, triggers); identifies known quirks (DualSense haptics Chrome-only, Switch Pro inconsistencies); generates a test matrix for QA sessions; suggests default deadzone values per controller type.
**Tools needed**: WebSearch, WebFetch, Read, Write.

### Agent 3: Recording Pipeline QA — Browser Video Output Validator
**Description**: Tests the recording pipeline across browsers and codecs. Validates output files are playable, have correct duration metadata, and meet platform requirements (TikTok, YouTube, Instagram).
**What it does**: Generates test scenarios for codec fallback chain (MP4/H.264 → WebM/VP9 → WebM/VP8); validates output files play in VLC, browser video elements, and social platform upload flows; checks duration metadata accuracy; monitors memory usage during recording; documents codec string compatibility per browser version.
**Tools needed**: Bash (ffprobe for video analysis), Read, WebSearch.

### Agent 4: Scene Composer — AI Scene Layout Assistant
**Description**: Helps users compose visually balanced scenes by suggesting character placement, prop arrangement, and camera settings based on the selected background and number of characters.
**What it does**: Analyzes background image composition (walkable area, visual weight); suggests character starting positions for 1-4 characters; recommends prop placement for set dressing; suggests camera mode and zoom level based on scene complexity; generates scene JSON presets for common scenarios (dialogue, chase, confrontation, group shot).
**Tools needed**: Read, image analysis.

### Agent 5: Sprint Planner — Phase Decomposition Agent
**Description**: Breaks down each development phase into actionable sprint-sized tasks with dependencies, estimates, and acceptance criteria. Monitors progress and flags risks.
**What it does**: Takes a phase from the roadmap and decomposes it into 1-3 day tasks; identifies inter-task dependencies; estimates effort using the T-shirt sizing from the PM analysis; generates GitHub issues or task list format; tracks progress against phase success criteria; flags when scope creep introduces tasks not in the phase plan.
**Tools needed**: Read, Write, Bash (git log for progress tracking).

### Agent 6: Phaser Migration Scout — v4 Readiness Monitor
**Description**: Monitors the Phaser v4 ecosystem and evaluates migration readiness. Tracks plugin compatibility, Vue template availability, and breaking changes relevant to Virtual Stage's codebase.
**What it does**: Checks npm for v4-compatible versions of rex plugins, merged-input, and Phavuer; searches for official Vue 3 + Phaser 4 templates; analyzes Virtual Stage's Phaser API usage against v4 breaking changes list; produces a quarterly "migration readiness report" with go/no-go recommendation; estimates migration effort.
**Tools needed**: WebSearch, WebFetch, Grep, Read.

---

## 11. Expert Analysis Documents

All expert analyses are available for detailed reference:

| Expert | Document | Key Contributions |
|--------|----------|-------------------|
| Research Analyst | `expert-research-analyst.md` | Competitive gap analysis (8 tools), Phaser v3 vs v4 decision, MP4 direct recording discovery, authenticity premium insight |
| Business Analyst | `expert-business-analyst.md` | 85 functional + 21 non-functional requirements, 8 structured use cases, stakeholder mapping, traceability matrix |
| Product/UX | `expert-product-ux.md` | 4 personas, 4 user journeys, feature prioritization (P0/P1/P2), information architecture, 10 UX recommendations, accessibility considerations |
| Architect | `expert-architect.md` | System diagram, 10 components with TypeScript interfaces, data model schemas, 6 ADRs, Firebase infrastructure design |
| Security/Compliance | `expert-security-compliance.md` | STRIDE threat model (7 components), 18 security requirements, 4 issues in architect's Firebase rules, corrected rules, GDPR checklist |
| Project Manager | `expert-project-manager.md` | 6 phases with deliverables and estimates, critical path, 10 risks, team requirements, 10 recommendations, scope cut/add guidance |

---

*Generated by the Virtual Stage project pipeline on 2026-04-12. 6 expert agents executed across 4 waves. 2 knowledge briefs produced. Full scale output.*
