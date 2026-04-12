# Expert Analysis: Project Manager -- Virtual Stage

**Project**: Virtual Stage
**Date**: 2026-04-12
**Agent**: Project Manager
**Input Sources**: intake.md, expert-research-analyst.md, expert-business-analyst.md, expert-product-ux.md, expert-architect.md, expert-security-compliance.md

---

## Project Overview

- **Scope**: Build a browser-based creative tool where users control pixel art characters with gamepads, speak dialogue via microphone, and record scenes as downloadable video. The application is a dual-framework system (Vue 3 shell + Phaser 3.90.0 canvas) communicating via a typed EventBridge, with five shared services (GamepadService, RecordingService, AudioService, SceneStateService, AssetService). The full scope includes single and multi-character puppeteering, a recording pipeline with codec negotiation, scene management with save/load, and an optional Firebase cloud layer. 85 functional requirements and 21 non-functional requirements have been identified. 4 security issues in the architect's Firebase rules require fixing before cloud features ship.

- **Complexity**: **Medium-High**
  - *Technical complexity*: Dual-framework integration (Vue + Phaser) with strict boundary enforcement. Real-time gamepad polling, audio capture, and video recording all running simultaneously in a browser tab. Cross-browser codec negotiation. State machine-driven character animation.
  - *Domain complexity*: First-mover in an unoccupied niche -- no existing product to reference for UX patterns. Gamepad-as-creative-input is a novel paradigm with no established conventions.
  - *Integration complexity*: Moderate. All browser-native APIs (Gamepad, MediaRecorder, Web Audio, captureStream). Firebase is additive and isolated. No third-party service dependencies for MVP.
  - *Scope complexity*: 6 phases with clear phase gates. MVP is well-defined (Phase 3). Post-MVP scope (Phases 4-6) is substantial but deferrable.

- **Recommended Approach**: **Iterative with phase gates**. Each phase produces a demonstrable, testable increment. Phases 1-3 are strictly sequential (each depends on the prior). Phases 4-6 have some parallelism potential. Adopt a "build the core loop first" strategy: the critical path runs through gamepad input -> character response -> canvas rendering -> video recording -> download. Every phase should be shippable to early testers. Continuous integration from Phase 1. Manual QA supplemented by unit tests for services and Firebase rules.

---

## Phases

### Phase 1: Foundation -- Phaser + Vue Shell with Gamepad Character Control

**Goal**: Establish the dual-framework architecture, render one character on a stage, and control it with a gamepad.

**Deliverables**:

| Deliverable | Effort | Notes |
|---|---|---|
| Vite + Vue 3 + TypeScript project scaffold from `phaserjs/template-vue-ts` | S | Existing official template; adapt to project structure |
| PhaserCanvas.vue component (mount/destroy lifecycle with `shallowRef`/`markRaw`) | S | Critical Vue-Phaser boundary piece |
| EventBridge (typed mitt instance + full EventMap type definitions) | S | Foundation for all cross-framework communication |
| GamepadService (polling loop, radial deadzone, button state tracking, debounce) | M | Standalone service, framework-agnostic. Must support up to 4 pads from day one |
| StageScene (Phaser scene with background rendering, one character slot) | M | BootScene for asset loading, StageScene for gameplay |
| Character entity (sprite + state machine: IDLE, WALK, RUN, JUMP, ACTION) | L | 8-state FSM with transition rules, directional animation, lock mechanism |
| CharacterController system (gamepad input -> character commands, default bindings) | M | Maps left stick to move, right stick to look-at, face buttons to actions |
| AssetService + manifest.json (one character sprite sheet + one background) | M | Data-driven loading. Source one LPC-compatible knight from itch.io with full animation set |
| Top Bar Vue component (scene name placeholder, gamepad status indicator) | S | Minimal UI shell |
| No-gamepad detection with guidance message (REQ-NF013) | S | Prominent banner when no gamepad detected |
| TypeScript strict mode, ESLint config, Prettier, project structure enforcement | S | `src/vue/`, `src/phaser/`, `src/bridge/`, `src/services/`, `src/types/` boundary rules |
| CI pipeline (build + lint + `npm audit`) | S | GitHub Actions. SEC-009 compliance from day one |

**Dependencies**: None (first phase).

**Milestone**: A user can open the app in Chrome, connect an Xbox/PS5 controller, see a pixel art character on a background, and control it with both joysticks and face buttons. Character walks, runs, jumps, attacks, crouches, blocks, and transitions between states correctly. Input latency is under 50ms.

**Success Criteria**:
- Character responds to all mapped gamepad inputs with correct state transitions
- No-gamepad state shows clear guidance
- Vue and Phaser have zero direct imports between their directories
- CI passes: build, lint, npm audit

**Estimated Duration**: 2.5-3.5 weeks

---

### Phase 2: Actions + Audio -- Full Action System, Emote Wheel, Microphone Capture

**Goal**: Complete the character's expressive range and add microphone capture with visual feedback.

**Deliverables**:

| Deliverable | Effort | Notes |
|---|---|---|
| Emote wheel system (LB/RB modifiers + face buttons = 8 emotes) | M | Radial UI rendered in Phaser near character when bumper held (UX R5) |
| Complete state machine (add CROUCH, EMOTE, DEAD states + transitions) | M | Death + get-up flow (D-pad Down), crouch hold (LT), block hold (RT) |
| Emote animations (wave, shrug, laugh, cry, celebrate, point, sit, fall) | M | May need additional sprite sheet frames or creative reuse of existing |
| AudioService (getUserMedia + GainNode + AnalyserNode + MediaStreamDestination) | M | SEC-001: explicit user action to init mic. SEC-003: cleanup on unload |
| Audio controls Vue component (gain slider, VU meter, mic status indicator) | M | SEC-002: persistent visual indicator when mic active |
| Privacy notice before first mic activation (SEC-016) | S | Dismissible but not bypassable |
| Keyboard fallback for basic controls (WASD + arrows + number keys) | M | UX R3: integrated into GamepadService, same action events |
| Costume cycling (D-pad Up) and run lock toggle (L3) | S | Low effort, high expressiveness gain |
| Gamepad connection/disconnection event handling with visual feedback | S | Character freezes gracefully on disconnect (REQ-NF014) |

**Dependencies**: Phase 1 complete (character entity, GamepadService, EventBridge).

**Milestone**: Character has full expressive range -- all 8 states, 8 emotes via wheel, costume cycling. Microphone captures audio with user-controlled gain and visual level indicator. Keyboard users can perform basic character control. Privacy notice appears before first mic use.

**Success Criteria**:
- All 8 emotes triggerable via bumper + face button combination
- Mic captures audio (verified via VU meter) without auto-initialization
- Keyboard fallback controls a character through all basic actions
- Gamepad disconnect during operation does not crash the application

**Estimated Duration**: 2-3 weeks

---

### Phase 3: Recording Pipeline (MVP) -- Record + Preview + Download

**Goal**: Complete the core value proposition: record a canvas + audio performance and download it as a video file. This is the MVP gate.

**Deliverables**:

| Deliverable | Effort | Notes |
|---|---|---|
| RecordingService (codec detection, stream composition, MediaRecorder lifecycle) | L | Codec fallback chain: MP4/H.264 -> WebM/VP9 -> WebM/VP8. Runtime `isTypeSupported()` negotiation |
| Countdown overlay (3-2-1 in Phaser, fires before MediaRecorder starts) | S | Countdown must NOT appear in recorded output (UX R2) |
| Recording controls Vue component (start/stop button, status label) | M | Gamepad Start button integration. Auto-hide UI on record start (UX R2) |
| Recording indicator (red dot + timer in Vue overlay, outside canvas capture) | S | UX R10: visible to user, absent from recording |
| Preview modal (video playback, format/size info, download, discard, record-again) | M | `<video>` element with Blob URL. Show format and estimated size on download button |
| Video download (MP4 on Chrome/Edge, WebM fallback) | M | Blob URL creation + anchor download + revokeObjectURL (SEC-014) |
| Tab visibility detection + auto-pause during recording (Page Visibility API) | M | UX R6: auto-pause on tab switch, banner on restore, no silent corruption |
| Memory/duration limit enforcement (450MB / 10 min defaults) | S | Warning at threshold, auto-stop at limit |
| Pre-recording status check (mic status, format indicator) | S | UX R2: 1-second status bar before countdown |
| UI auto-hide when recording starts | S | UX R2: auto-hide Vue panels, toast notification |
| `beforeunload` handler for stream cleanup | S | SEC-003: stop all tracks on navigate away / tab close |
| Content Security Policy headers configuration (firebase.json) | S | SEC-006, SEC-015: full CSP, X-Frame-Options, Permissions-Policy |
| ESLint security rules (`eslint-plugin-security`, `vue/no-v-html`) | S | SEC-008, SEC-009 |

**Dependencies**: Phase 2 complete (AudioService providing MediaStream, full character system for meaningful recordings).

**Milestone**: **MVP COMPLETE**. A user can control a pixel art character with a gamepad, speak dialogue, record the performance, preview it, and download an MP4/WebM video file. The complete input-to-output pipeline works end-to-end.

**Success Criteria**:
- User records a 30-second scene and downloads a playable MP4 (Chrome) or WebM (Firefox)
- Recording has synchronized audio and video
- Tab switch during recording triggers auto-pause (not silent corruption)
- Memory stays under 500MB for a 10-minute recording
- CSP headers deployed and validated
- No `v-html` usage in codebase (ESLint enforced)

**Estimated Duration**: 3-4 weeks

---

### Phase 4: Multi-Character + Scenes -- Multiple Gamepads, Backgrounds, Camera, Save/Load

**Goal**: Transform the single-character MVP into a multi-character stage with scene composition and persistence.

**Deliverables**:

| Deliverable | Effort | Notes |
|---|---|---|
| Multi-character support (up to 4 characters on stage simultaneously) | M | Extend StageScene to manage 4 character slots |
| Gamepad-to-character assignment UI (Character Panel with dropdown + "press to identify") | M | UX R7: identify controller flow, colored outline feedback |
| Multi-gamepad simultaneous input handling | M | Each gamepad drives only its assigned character. Conflict prevention |
| Character Panel Vue component (add/remove characters, select types, assign pads) | M | 3+ character types from asset library |
| Background selector (Scene Panel with thumbnail grid, 5+ backgrounds) | M | Immediate canvas update on selection |
| Prop placement system (browse, place, drag, resize, flip, layer assignment) | L | Depth sorting by Y position. Foreground/background layers |
| Camera modes (Fixed, Follow, Free Pan) + zoom control | M | CameraController system with lerp follow, pan input |
| Scene Panel Vue component (backgrounds, props browser, camera controls) | M | Tabbed sidebar layout per UX information architecture |
| SceneStateService (JSON serialization/deserialization) | M | Full scene state: background, characters, props, camera |
| Save to localStorage + load from localStorage | M | Named saves, list/load/delete. REQ-NF016: idempotent save/load |
| Scene export/import as JSON file | S | File API download + file picker upload with validation |
| Auto-save every 30 seconds + session restore on load | M | UX R9: rolling autosave slot, restore banner on load |
| Scene JSON schema validation (Zod) | M | SEC-007: reject unknown fields, strip prototype pollution keys, enforce string limits |
| Additional asset sourcing (3 character types, 5 backgrounds, 15 props) | L | From itch.io CC0/CC-BY-SA packs. Attribution data in manifest |
| Attribution panel (credits/licenses for all bundled assets) | S | SEC-012: accessible from main UI at all times |
| Asset lazy loading (only current scene assets loaded) | M | REQ-NF020: on-demand loading with progress indicator |

**Dependencies**: Phase 3 complete (recording pipeline works, so multi-character recording works automatically). Asset sourcing can begin during Phase 3.

**Milestone**: Multi-player collaborative recording works -- 4 friends with 4 gamepads controlling 4 characters on a composed stage, recorded as a single video. Scenes can be saved, loaded, exported, and shared as JSON files.

**Success Criteria**:
- 4 simultaneous gamepads control 4 independent characters without input crosstalk
- Gamepad assignment via "press to identify" flow completes in under 30 seconds
- Scene save/load is idempotent (save -> load -> save produces identical JSON)
- Imported scene JSON with malicious payloads (`__proto__`, oversized strings) is rejected
- 60fps maintained with 4 characters + 15 props on mid-range hardware (REQ-NF001)

**Estimated Duration**: 4-6 weeks

---

### Phase 5: Polish + UX -- Onboarding, Quick Start, Props Refinement, Quality of Life

**Goal**: Make the tool approachable for new users and polished for returning users. Address discoverability and first-time experience.

**Deliverables**:

| Deliverable | Effort | Notes |
|---|---|---|
| Onboarding overlay (interactive tutorial: move, look, act, record, download) | L | UX R4: under 2 minutes, skippable, ends with real artifact. No-gamepad variant |
| Quick Start mode (random background, auto-place characters per detected gamepads) | M | UX R8: under 30 seconds to "ready to record" for party scenario |
| Gamepad controls reference overlay (visual diagram, toggle from Help or F1) | M | UX R1: brief flash on first connection, persistent toggle |
| Action Binding Editor (remap gamepad buttons, live detection, persist to localStorage) | M | Listen mode, conflict detection, reset to defaults |
| Action hints on idle (floating icons near character during non-recording) | S | UX R5: subtle, suppressed during recording |
| Speech bubbles (text overlays above characters) | M | REQ-F082: triggered by user input or mic activity |
| Scene transitions (fade, wipe between backgrounds) | M | Phaser v3 camera fade/shader effects. Could leverage future v4 Filter system |
| Settings modal (audio device, recording format, deadzone, display options) | M | Consolidates all user preferences |
| Fullscreen mode suggestion on first recording | S | UX R6: non-blocking tooltip |
| Unsupported browser detection (Safari, mobile) with clear message | S | REQ-NF010 |
| Accessibility pass: WCAG AA contrast, semantic HTML, focus management, ARIA labels | M | Screen reader support for Vue UI. `prefers-reduced-motion` respect |
| Performance optimization pass (memory profiling, frame rate validation) | M | Verify 60fps with full scene, verify recording memory bounds |
| Source map exclusion from production builds | S | SEC-018 |

**Dependencies**: Phase 4 complete (multi-character and scene system exist for onboarding to teach, Quick Start to use). Can begin UX design work during Phase 4.

**Milestone**: A first-time user with no prior knowledge can complete the onboarding, create a scene, and download a recording within 5 minutes. A party of 4 can be set up and recording within 2 minutes via Quick Start.

**Success Criteria**:
- New user completes onboarding in under 2 minutes (timed usability test)
- Quick Start produces a ready-to-record 4-character stage in under 30 seconds
- All Vue UI elements meet WCAG 2.1 AA contrast ratios
- No `console.log` of sensitive data in production build
- Settings persist across sessions via localStorage

**Estimated Duration**: 3-5 weeks

---

### Phase 6: Firebase + Cloud -- Auth, Cloud Save, Recording Storage, Sharing

**Goal**: Add optional cloud features for cross-device access and content sharing. Application remains fully functional without authentication.

**Deliverables**:

| Deliverable | Effort | Notes |
|---|---|---|
| Firebase project setup (Auth, Firestore, Cloud Storage, Hosting) | M | Region selection, billing alerts, App Check configuration |
| Firebase Auth with Google sign-in | M | SEC-004: corrected Firestore create rules (ownerId validation) |
| Corrected Firestore security rules (per Security analysis ISSUE-1, ISSUE-3, ISSUE-4) | M | Owner-only create/update, immutable ownerId, string length limits |
| Corrected Cloud Storage security rules (per Security analysis ISSUE-2) | M | SEC-005: no public read. Signed URLs for sharing via Cloud Function |
| Cloud scene save/load (Firestore) | M | Sync with existing SceneStateService. Public/private/shared scenes |
| Cloud recording upload (Cloud Storage, 500MB limit per file) | L | Upload progress indicator. Per-user quota enforcement (SEC-011, 2GB) |
| Shareable links for recordings (Cloud Function generating signed URLs) | M | Time-limited, revocable. Firestore `isPublic` flag |
| Firebase App Check (reCAPTCHA Enterprise provider) | M | SEC-010: restrict API access to legitimate app |
| Firebase Hosting deployment pipeline | M | Preview channels for PRs. Production gated behind approval |
| Firebase security rules unit tests (@firebase/rules-unit-testing + Emulator Suite) | L | All test cases from Security analysis. Run before every rules deploy |
| Privacy policy page | M | GDPR: data collected, processing, retention, deletion rights, legal basis |
| Account deletion flow (user data + Firestore docs + Storage files) | M | GDPR right to deletion |
| Terms of Service (content policy, abuse reporting, DMCA process) | M | Required before enabling public cloud storage |
| Cookie consent for Firebase Analytics (if enabled) | S | ePrivacy Directive for EU users |

**Dependencies**: Phase 3+ complete (recording pipeline and scene system). Security rule fixes are blockers for this phase -- do not deploy Firebase without the corrected rules from the Security analysis.

**Milestone**: Users can sign in with Google, save scenes to the cloud, upload recordings, and share them via links. All cloud operations are secured with corrected Firebase rules, validated by automated tests.

**Success Criteria**:
- Authenticated user saves scene on one device, loads it on another
- Unauthenticated user cannot read/write protected Firestore paths (rules test suite passes)
- Malicious create with forged ownerId is rejected (ISSUE-1 fix verified)
- Public recording sharing works via signed URL, not raw storage path (ISSUE-2 fix verified)
- ownerId cannot be changed on update (ISSUE-3 fix verified)
- Scene names exceeding 200 chars are rejected by rules (ISSUE-4 fix verified)
- App functions identically when not signed in (local-first principle preserved)

**Estimated Duration**: 4-6 weeks

---

## Critical Path

The minimum timeline is determined by the following sequential dependency chain:

```
Phase 1: Foundation (2.5-3.5 wk)
    │
    ▼  [Character + GamepadService + EventBridge required]
Phase 2: Actions + Audio (2-3 wk)
    │
    ▼  [AudioService MediaStream + full character system required]
Phase 3: Recording Pipeline / MVP (3-4 wk)
    │
    ▼  [MVP complete — recording works end-to-end]
Phase 4: Multi-Character + Scenes (4-6 wk)
    │
    ▼  [Multi-character + scene system required for onboarding content]
Phase 5: Polish + UX (3-5 wk)
    │
    ▼  [Core features stable for cloud integration]
Phase 6: Firebase + Cloud (4-6 wk)
```

**Critical path items within phases**:

1. **Phase 1**: Character state machine and GamepadService are on the critical path. The EventBridge must be designed correctly on the first pass -- changing the event contract later propagates to every component. Asset sourcing (finding a suitable LPC knight sprite sheet) could block if no pack has all 8 required animation states.

2. **Phase 2**: AudioService is the bottleneck -- the RecordingService in Phase 3 cannot start without a working audio stream. Emote animations may require additional sprite sheet work if the sourced pack lacks emote frames.

3. **Phase 3**: Codec detection and the MediaRecorder pipeline are the highest-risk items. If `captureStream` + `MediaRecorder` does not produce clean output on the first integration attempt, debugging is time-intensive because failures are only visible in the recorded output (not in real-time).

4. **Phase 4**: Asset sourcing (3 characters, 5 backgrounds, 15 props) is the most likely schedule risk. Sprite sheets from different itch.io packs may have inconsistent formats requiring adaptation work.

5. **Phase 5**: Onboarding design depends on the final feature set from Phase 4. Design work can start during Phase 4 but implementation must wait.

6. **Phase 6**: Security rule fixes (4 issues from Security analysis) are on the critical path. Do not ship Firebase without corrected rules and passing test suite.

**Minimum timeline to MVP (Phase 3 complete)**: 7.5-10.5 weeks (roughly 2-2.5 months)

**Minimum timeline to full scope (Phase 6 complete)**: 19-27.5 weeks (roughly 5-7 months)

---

## Risk Assessment

| # | Risk | Impact | Probability | Mitigation |
|---|------|--------|-------------|------------|
| R1 | **Vue-Phaser lifecycle conflicts** -- Phaser game objects wrapped in Vue reactivity proxies cause memory leaks, GC thrash, or crash. Phaser scene destruction on Vue unmount fails or leaves dangling references. | High | Medium | Enforce `shallowRef`/`markRaw` for all Phaser references. Never make Phaser objects reactive. Build PhaserCanvas.vue lifecycle tests early. Architect has specified the pattern -- follow it precisely. |
| R2 | **Background tab throttling corrupts recordings** -- User switches tabs during recording; requestAnimationFrame drops to ~1fps; recorded output is choppy with no recovery. | High | High | Page Visibility API auto-pause (Phase 3 deliverable). Fullscreen mode suggestion. Pre-recording warning. This is the highest-probability UX risk -- build and test the mitigation before any user testing. |
| R3 | **Codec fragmentation across browsers** -- MediaRecorder codec support varies. Direct MP4 recording may fail on some Chrome versions. Firefox only supports VP8/WebM. Edge cases in codec strings. | Medium | Medium | Runtime `isTypeSupported()` negotiation with fallback chain (already designed in RecordingService). Test on Chrome, Edge, Firefox on every recording-related change. Accept that Firefox output is WebM-only. |
| R4 | **Phaser v3 EOL vulnerability** -- Phaser 3.90.0 is the final v3 release. Any security vulnerability discovered in v3 has no upstream fix path. v4 ecosystem may not stabilize for 3-6 months. | Medium | Low | Monitor Phaser security discussions. The EventBridge pattern minimizes Phaser-version coupling. Plan v4 migration for post-Phase 6 once rex plugins and Vue templates confirm v4 support. Do not depend on Phaser renderer internals. |
| R5 | **Memory pressure during long recordings** -- Blob accumulation during 10-minute recordings approaches browser memory limits (~1GB Chrome). Combined with Phaser scene memory, total usage may cause tab crash. | High | Medium | RecordingService enforces 450MB blob limit and 10-minute duration cap. Memory warning events at threshold. Profile actual memory usage in Phase 3 with a full recording session. If memory is tighter than expected, reduce default limits. |
| R6 | **Gamepad API inconsistency across controllers** -- DualSense (PS5), Switch Pro, and third-party controllers may have non-standard mappings. Some controllers report as "standard" but have swapped buttons. | Medium | Medium | Default to Standard Gamepad layout (W3C). Button remapping in Phase 5 is the definitive fix. During Phase 1, test with at least Xbox and one non-Xbox controller. Document known mapping quirks. |
| R7 | **Sprite sheet format inconsistency across asset packs** -- itch.io assets use varying frame sizes (16x16, 32x32, 64x64), padding, and animation naming. Integrating 3+ character types from different sources requires adaptation. | Medium | High | Standardize on 64x64 LPC format as the canonical format. AssetService handles frame size configuration per character. Budget extra time in Phase 4 for asset integration. Consider commissioning a single consistent set if free assets prove too inconsistent. |
| R8 | **Firebase security misconfiguration** -- The architect's original rules have 4 identified issues (forged ownerId, public recording reads, mutable ownership, no string limits). Shipping cloud features with unfixed rules exposes user data. | High | Low (if mitigations followed) | Security analysis provides corrected rules verbatim. Write comprehensive rules test suite BEFORE deploying. Block Phase 6 deployment on all security tests passing. Never deploy rules without running the Firebase Emulator test suite. |
| R9 | **Scope creep via "nice-to-have" features** -- Speech bubbles, scene transitions, ffmpeg.wasm conversion, chroma key backgrounds, multi-mic support, and custom asset uploads are all discussed but not in MVP. Temptation to add them early. | Medium | High | Strict phase gates. MVP is Phase 3: one character, one background, record, download. Features not in the current phase do not get worked on. Use the priority matrix from UX (P0/P1/P2) as the arbiter. If it is P2, it waits. |
| R10 | **veadotube full version competition** -- veadotube's upcoming full version may add scene editor and gamepad-driven spatial movement, closing the competitive gap. s&box (April 28, 2026 launch) may attract machinima creators. | Low | Low | Research analyst confirmed veadotube full version focuses on node-based animation, not gamepad puppeteering. Virtual Stage's core differentiator (spatial gamepad control + recording) is not on veadotube's published roadmap. Monitor quarterly. Ship MVP before competitive landscape shifts. |

---

## Team Requirements

This analysis assumes a solo developer or very small team (1-2 developers), which is consistent with the project's scope as an indie creative tool.

| Role | When Needed | Responsibility | Notes |
|------|-------------|----------------|-------|
| **Full-Stack Developer (Lead)** | Phase 1-6 | Vue 3, TypeScript, Phaser integration, service architecture, recording pipeline, Firebase. Owns the entire codebase. | Must be comfortable with both Vue 3 Composition API and Phaser's game loop paradigm. This is the one mandatory role. |
| **Pixel Art Asset Curator** | Phase 1 (start), Phase 4 (heavy) | Source, evaluate, and adapt open-license sprite sheets and backgrounds. Ensure format consistency (64x64, 4-direction, JSON animation defs). Track attribution/licensing. | Does not need to create original art. Needs familiarity with LPC format, itch.io asset ecosystem, and sprite sheet tools (Aseprite or equivalent for adaptation). Can be the lead developer wearing a second hat, or a part-time contributor. |
| **UX Designer** | Phase 4-5 | Onboarding flow design, Quick Start flow, sidebar layout refinement, accessibility audit, usability testing. | Optional but high-value. Without this role, the developer designs the UX based on the Product/UX expert analysis (which is very detailed). If available, start during Phase 4. |
| **QA / Tester** | Phase 3+ (periodic) | Cross-browser recording testing (Chrome, Edge, Firefox). Multi-gamepad testing. Firebase security rules penetration testing (Phase 6). | Can be informal -- friends or community members testing with different controllers and browsers. Structured testing needed for the recording pipeline and Firebase rules. |

**Solo Developer Feasibility Assessment**: This project is feasible for a solo developer. The Product/UX analysis and Architecture analysis are detailed enough to serve as a de facto design document and technical specification. The biggest solo-developer risks are: (1) testing the multi-gamepad flow requires physically having 4 controllers, (2) cross-browser testing is time-consuming, and (3) asset curation competes with coding time. A second contributor focused on asset curation would meaningfully accelerate Phases 1 and 4.

---

## Milestones & Timeline

Estimates assume a solo developer working approximately 25-35 productive hours per week. All durations are ranges reflecting uncertainty.

| Milestone | Phase | Target Week | Success Criteria |
|-----------|-------|-------------|-----------------|
| **M1: Character Alive** | Phase 1 | Week 3 | Character rendered on background, controlled by gamepad with correct state transitions. CI pipeline passing. Dual-framework architecture verified (no cross-boundary imports). |
| **M2: Full Expression** | Phase 2 | Week 6 | All 8 character states, 8 emotes, keyboard fallback, microphone capturing with VU meter. Privacy notice before mic activation. |
| **M3: MVP -- Record & Download** | Phase 3 | Week 9-10 | End-to-end: control character -> speak -> record -> preview -> download MP4. Tab visibility protection working. CSP headers deployed. This is the first externally shareable milestone. |
| **M4: Party Mode** | Phase 4 | Week 15-16 | 4 gamepads, 4 characters, scene composition with backgrounds/props, save/load. Asset library populated (3 chars, 5 bgs, 15 props). Scene JSON validated against Zod schema. |
| **M5: First-Time Ready** | Phase 5 | Week 19-20 | Onboarding completes in under 2 minutes. Quick Start works. Settings persist. Accessibility pass complete. Performance validated at 60fps with full scene. |
| **M6: Cloud Launch** | Phase 6 | Week 24-26 | Firebase deployed with corrected security rules (all 4 issues fixed). Rules test suite passes. Auth, cloud save, recording upload, sharing links all functional. Privacy policy and ToS published. |

**Timeline Visualization**:

```
Week:  1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26
       |---Phase 1---|---Phase 2---|-----Phase 3------|--------Phase 4---------|----Phase 5----|----Phase 6----|
                   M1           M2              M3                          M4              M5              M6
                                                 ^
                                           MVP GATE
```

**Important caveats on timeline**:
- These are estimates for a solo developer at a sustainable pace, not a sprint. Burnout on a 6-month project is a real risk.
- Asset sourcing is the wildcard. If suitable LPC-compatible packs exist with all needed animations, the timeline holds. If significant sprite sheet adaptation is needed, add 1-2 weeks to Phases 1 and 4.
- Cross-browser testing and debugging can expand Phase 3 by 1-2 weeks if codec edge cases surface.
- The Phase 4 -> Phase 5 boundary is the best place to cut scope if behind schedule (see Recommendations).

---

## Recommendations

### R1: Build the recording pipeline proof-of-concept in Phase 1, not Phase 3

**Rationale**: The recording pipeline (captureStream + MediaRecorder + codec negotiation) is the highest-risk technical component. If it does not work as expected, the entire product concept is at risk. Do not wait until Phase 3 to discover this.

**Action**: In Phase 1, as a parallel spike (1-2 days), create a minimal proof-of-concept: capture the Phaser canvas with `captureStream(30)`, record 10 seconds with MediaRecorder using the codec fallback chain, and verify the output plays correctly. This validates the core technical assumption before investing in the full character system. If the PoC fails, the project has a fundamental constraint to design around early.

### R2: Strict phase gates -- do not start Phase N+1 until Phase N's success criteria are met

**Rationale**: The phases are designed with clear success criteria for a reason. Scope creep and "just one more feature" are the most probable causes of project failure on a solo project.

**Action**: At the end of each phase, run the success criteria as a checklist. If any criterion is not met, fix it before moving on. This applies especially to M3 (MVP) -- if the recording pipeline has quality issues, fix them before adding multi-character support.

### R3: Prioritize asset sourcing early -- start during Phase 1

**Rationale**: Finding open-license pixel art packs with complete animation sets (all 8 states + 8 emotes + costumes) is harder than it appears. The LPC system has many characters but the animation variety may not cover all emotes. itch.io packs vary in quality and format.

**Action**: Begin asset scouting in Phase 1, aiming to have:
- Phase 1: 1 knight character with walk/run/idle/jump/attack animations. 1 background.
- Phase 2: Emote animations for the knight (may require a different pack or creative reuse).
- Phase 4: 3 character types, 5 backgrounds, 15 props -- all sourced, format-validated, and attribution-documented.

Create a spreadsheet tracking: pack name, source URL, license, frame size, available animations, adaptation needed.

### R4: Test with real gamepads from day one -- do not rely on keyboard simulation

**Rationale**: Gamepad API behavior in real browsers with real controllers is different from what documentation suggests. DualSense trigger values, Switch Pro button mappings, and Xbox Elite paddle buttons all have quirks. The product is built around the gamepad experience.

**Action**: Have at least 2 different controllers available from Phase 1 (ideally Xbox + DualSense). Test gamepad input on every commit that touches GamepadService or CharacterController.

### R5: If behind schedule, cut scope in this order

The phase structure supports graceful degradation. If the timeline is slipping:

1. **Cut first**: Phase 6 (Firebase/Cloud). The tool works entirely locally. Cloud features are a convenience, not the value proposition. Ship local-only.
2. **Cut second**: Phase 5 nice-to-haves -- speech bubbles and scene transitions. Keep onboarding and Quick Start (they drive retention).
3. **Simplify**: Phase 4 prop system. Reduce from full drag/resize/layer to simple placed sprites at fixed size. Keep multi-character and save/load.
4. **Do not cut**: Phase 3 recording pipeline quality. A recording tool that produces bad recordings has negative word-of-mouth.
5. **Do not cut**: Phase 2 keyboard fallback. It broadens the audience at low cost.

### R6: If ahead of schedule, add features in this order

1. **Add first**: MP4 direct recording with quality presets (Low/Medium/High bitrate). Immediately improves the output for TikTok/YouTube creators.
2. **Add second**: Chroma key / transparent background for OBS streamers. High value for the streaming persona, moderate effort.
3. **Add third**: Aseprite JSON direct import. Opens the door for the pixel art creator community to bring their own characters.
4. **Add fourth**: Director mode (character state display, take numbering, scene markers). High value for the Indie Filmmaker persona.

### R7: Establish a competitive monitoring cadence

**Rationale**: The research analyst confirmed no direct competitor exists today, but the veadotube full version (2026 release) and s&box (April 28, 2026) could shift the landscape.

**Action**: Set a quarterly check (July 2026, October 2026, January 2027):
- Has veadotube full version added gamepad-driven spatial movement?
- Has any new browser-based creative tool entered the pixel art machinima space?
- Has Phaser v4 ecosystem matured enough for migration (rex plugins, Vue template)?

### R8: Plan the Phaser v4 migration but do not execute it before Phase 6

**Rationale**: Phaser 4.0.0 released 2 days ago. The v4 Filter system (Pixelate, Glow, Bloom, Vignette, Wipe) would be particularly valuable for scene transitions and visual effects. But the ecosystem needs 3-6 months to catch up.

**Action**: During Phase 5 or 6, evaluate: (1) Are rex plugins v4-compatible? (2) Is there a Vue 3 + Phaser 4 template? (3) Are there breaking changes in APIs Virtual Stage uses? If all three pass, plan a v4 migration as a Phase 7. The EventBridge architecture was specifically chosen to minimize this migration surface.

### R9: Write Firebase security rules tests BEFORE writing the rules

**Rationale**: The Security analysis found 4 issues in the architect's rules. Rules are notoriously easy to get wrong and catastrophic when wrong (user data exposure, resource abuse).

**Action**: In Phase 6, write the test suite first (test-driven security). Use `@firebase/rules-unit-testing` with the Firebase Emulator Suite. Test every case from the Security analysis checklist. Then write the corrected rules to make the tests pass. Never deploy rules that have not passed the full test suite.

### R10: Ship MVP (M3) to a small group of content creators for feedback before Phase 4

**Rationale**: The product occupies an unoccupied niche with no existing user expectations. Real user feedback on the gamepad-to-recording pipeline will surface UX issues that no expert analysis can predict. The multi-character and scene features (Phase 4) should be informed by real usage data.

**Action**: After M3, deploy to Firebase Hosting (or a simple static host) and share with 5-10 content creators who use gamepads. Collect feedback on: (1) Was the recording quality acceptable? (2) Did the gamepad controls feel intuitive? (3) What did you try to do that you could not? (4) Would you use this to make content? Incorporate findings into Phase 4 planning.

---

## Open Dependencies and Blockers

| Item | Blocking | Status | Action Needed |
|------|----------|--------|---------------|
| Sprite sheet sourcing (knight with 8+ animation states) | Phase 1 start | [HUMAN-INPUT-NEEDED] | Identify and download a suitable LPC or itch.io pack before Phase 1 coding begins |
| Monetization strategy decision | Phase 6 design | [HUMAN-INPUT-NEEDED] | Defines whether asset system needs paid asset delivery hooks. Does not block MVP. |
| Chroma key / transparent background decision | Phase 5 scope | [HUMAN-INPUT-NEEDED] | If yes, add to Phase 5 deliverables (M effort). High value for streamer persona. |
| Maximum recording duration decision | Phase 3 configuration | [HUMAN-INPUT-NEEDED] | 10 min cap may frustrate indie filmmakers. Recommend 10 min default, 15 min max configurable. |
| Keyboard fallback scope (full parity or subset?) | Phase 2 scope | [HUMAN-INPUT-NEEDED] | Recommend subset: movement + face button actions + recording toggle. Not emotes or camera. |
| Art asset licensing legal review (GPL on sprites) | Phase 4 | [HUMAN-INPUT-NEEDED] | GPL on art assets used in a free tool is likely fine, but formal review is prudent before expanding the asset library. |
| Security rule fixes (4 issues from Security analysis) | Phase 6 deploy | Documented | Must be implemented and tested before any Firebase deployment. Corrected rules provided in Security analysis. |

---

## Summary

Virtual Stage is a well-scoped, technically feasible project with a clear MVP gate at Phase 3 (~10 weeks for a solo developer). The first-mover advantage in the pixel art gamepad puppeteering niche is real and confirmed by competitive analysis. The primary risks are technical (Vue-Phaser lifecycle, recording pipeline fidelity, codec fragmentation) rather than market-based. The architecture is sound -- the EventBridge pattern and service layer provide clean separation that supports incremental delivery and future Phaser v4 migration.

The single most important recommendation is to validate the recording pipeline early (Phase 1 spike) and ship the MVP to real users for feedback before investing in multi-character and cloud features. If the core loop -- pick up gamepad, control character, speak, record, download -- delights users, everything else follows. If it does not, no amount of additional features will save the product.

**Total estimated effort**: 19-27.5 weeks (5-7 months) for full scope. 7.5-10.5 weeks (2-2.5 months) to MVP.

---

*End of Project Management Analysis*
