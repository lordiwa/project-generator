# Expert Analysis: Business Analyst — Virtual Stage

**Project**: Virtual Stage
**Date**: 2026-04-12
**Analyst**: Business Analyst Agent
**Input Sources**: intake.md, project-context.md, browser-game-recording-tech-brief.md, pixel-art-machinima-puppeteering-brief.md

---

## Stakeholders

### Primary Stakeholders

- **Content Creators (Primary Users)**: YouTubers and TikTokers producing gaming-adjacent content. Produce short animated skits (30s-5min). Familiar with gamepads. Value production speed over polish. Concerned with: ease of use, quick scene setup, fast recording-to-download turnaround, output format compatibility with YouTube/TikTok/Reels.

- **Streamers (Secondary Users)**: Twitch/YouTube Live streamers using virtual avatars. Require real-time pixel art puppets controlled by gamepad during streams. Concerned with: low-latency character response, OBS integration, visual consistency during extended sessions, no dropped frames.

- **Indie Filmmakers (Tertiary Users)**: Animation hobbyists and indie game developers producing longer-form animated shorts (5-30min). Need multi-character scenes, precise camera control, multiple takes. Concerned with: scene complexity, save/load reliability, recording duration limits, video quality.

### Secondary Stakeholders

- **Pixel Art Asset Creators**: Artists who create sprite sheets and backgrounds for use in the tool. May participate in a future marketplace. Concerned with: asset format standards (sprite sheet + JSON), attribution requirements for open-license assets, upload validation, display fidelity (no sub-pixel artifacts).

- **Development Team**: Engineers building and maintaining the application. Concerned with: Vue 3 + Phaser.js integration stability, Gamepad API cross-browser inconsistencies, MediaRecorder codec support, tech debt from dual-framework architecture, Phaser v3 vs v4 migration path.

- **Project Owner / Product Owner**: Decision-maker for feature prioritization and monetization strategy. Concerned with: competitive positioning in unoccupied niche, user acquisition, future monetization (premium assets, cloud tiers), maintaining the "free creative tool" value proposition during early growth.

### External Stakeholders

- **Browser Vendors (Chrome, Edge, Firefox)**: Indirect stakeholders whose API implementations (Gamepad API, MediaRecorder, Web Audio API, captureStream) constrain the feature set. Chrome/Edge are primary targets; Firefox is secondary with known limitations.

- **Content Platforms (YouTube, TikTok, Twitch)**: Indirect stakeholders whose format requirements (MP4 preferred, specific aspect ratios) influence recording output specifications.

- **Open-License Asset Communities (itch.io, LPC)**: Providers of initial art assets under GPL/CC-BY-SA/CC0 licenses. Compliance with attribution requirements is mandatory.

---

## Requirements

### Functional Requirements

#### Gamepad Input System

- **REQ-F001**: The system shall read input from gamepads connected via the browser Gamepad API, polling `navigator.getGamepads()` each animation frame. (Priority: **Must**)

- **REQ-F002**: The system shall support the Standard Gamepad layout (W3C mapping) covering Xbox, PlayStation (DualSense), and Switch Pro controllers without requiring user configuration. (Priority: **Must**)

- **REQ-F003**: The left joystick shall control character movement, where stick magnitude determines speed — below a walk threshold the character walks, above it the character runs. (Priority: **Must**)

- **REQ-F004**: The right joystick shall control look-at direction, causing the character to face the joystick direction independently of movement direction. Look-at direction shall be processed regardless of the current action state. (Priority: **Must**)

- **REQ-F005**: The system shall apply radial deadzone filtering with rescaling to both joysticks, with a default threshold of 0.15, to eliminate stick drift. (Priority: **Must**)

- **REQ-F006**: The system shall support up to 4 gamepads connected simultaneously, each tracked independently with its own deadzone filtering and button state. (Priority: **Must**)

- **REQ-F007**: The system shall debounce button presses to prevent unintended double-inputs on a single physical press. (Priority: **Must**)

- **REQ-F008**: The system shall implement an emote wheel system using bumper modifiers: LB + face button triggers emote set 1 (Wave, Shrug, Laugh, Cry), RB + face button triggers emote set 2 (Celebrate, Point, Sit, Fall). (Priority: **Must**)

- **REQ-F009**: The Start button on the gamepad shall start and stop video recording. (Priority: **Must**)

- **REQ-F010**: The Select/Back button on the gamepad shall toggle the visibility of the Vue UI overlay for clean recording. (Priority: **Must**)

- **REQ-F011**: D-pad Up shall cycle through available character costumes. (Priority: **Should**)

- **REQ-F012**: D-pad Down shall trigger the theatrical death animation on the active character. (Priority: **Should**)

- **REQ-F013**: L3 (left stick click) shall toggle run lock, causing the character to run at full speed regardless of stick magnitude until toggled off. (Priority: **Should**)

- **REQ-F014**: R3 (right stick click) shall reset the character's facing direction to match the current movement direction. (Priority: **Should**)

- **REQ-F015**: The user shall be able to remap gamepad button-to-action bindings via an in-app Action Binding Editor. The editor shall detect live button presses on the gamepad for remapping. (Priority: **Should**)

- **REQ-F016**: The deadzone threshold shall be user-configurable per gamepad, with a range of 0.00 to 0.25. (Priority: **Could**)

- **REQ-F017**: The GamepadService shall be a standalone service (not Phaser's built-in plugin) accessible from both Vue components and Phaser scenes. (Priority: **Must**)

#### Character System

- **REQ-F018**: Characters shall be rendered as animated sprites from sprite sheets with JSON animation definitions. (Priority: **Must**)

- **REQ-F019**: Characters shall support 4-directional sprites (down, up, left, right), with horizontal flip applied for left/right symmetry where applicable. (Priority: **Must**)

- **REQ-F020**: The character system shall implement a state machine with the following states: IDLE, WALK, RUN, JUMP, CROUCH, ACTION, EMOTE, DEAD. (Priority: **Must**)

- **REQ-F021**: State transitions shall follow defined rules — e.g., DEAD can only transition to IDLE via a "get up" action; non-interruptible actions (JUMP, ACTION, EMOTE) shall play to completion before accepting new input. (Priority: **Must**)

- **REQ-F022**: The A/Cross button shall trigger a Jump action. (Priority: **Must**)

- **REQ-F023**: The X/Square button shall trigger a primary theatrical attack (sword swing / punch). (Priority: **Must**)

- **REQ-F024**: The Y/Triangle button shall trigger a theatrical magic/ranged attack. (Priority: **Must**)

- **REQ-F025**: The B/Circle button shall trigger a context action / nod. (Priority: **Must**)

- **REQ-F026**: LT/L2 held shall place the character in the CROUCH state; releasing shall return to IDLE. (Priority: **Must**)

- **REQ-F027**: RT/R2 held shall trigger a shield block / defend pose; releasing shall return to IDLE. (Priority: **Must**)

- **REQ-F028**: All actions shall be theatrical only — no game mechanics, no hitboxes, no damage calculations. (Priority: **Must**)

- **REQ-F029**: Non-active characters (not assigned to a gamepad, or whose gamepad is idle) shall hold their last pose and play an idle animation. (Priority: **Must**)

- **REQ-F030**: The system shall support multiple character types (e.g., knight, mage, rogue) each with their own sprite sheet and action animation set. (Priority: **Should**)

- **REQ-F031**: The character system shall support a "get up" action that transitions from the DEAD state back to IDLE. (Priority: **Should**)

#### Recording Pipeline

- **REQ-F032**: The system shall capture the Phaser canvas output via `canvas.captureStream(30)` at 30 frames per second. (Priority: **Must**)

- **REQ-F033**: The system shall combine the canvas video stream and the microphone audio stream into a single MediaStream for recording. (Priority: **Must**)

- **REQ-F034**: The system shall use the MediaRecorder API to record the combined stream, producing WebM output with VP9 video and Opus audio codecs as the primary format. (Priority: **Must**)

- **REQ-F035**: The system shall detect codec support at runtime via `MediaRecorder.isTypeSupported()` and fall back to VP8+Opus if VP9 is unavailable. (Priority: **Must**)

- **REQ-F036**: The recording flow shall include a 3-2-1 visual countdown before recording begins. (Priority: **Must**)

- **REQ-F037**: After stopping a recording, the system shall present a preview modal that plays back the recorded video before the user commits to saving or discarding. (Priority: **Must**)

- **REQ-F038**: The user shall be able to download the recorded video as a file (WebM format). (Priority: **Must**)

- **REQ-F039**: The system shall display a recording indicator (red dot + elapsed timer) in the top bar while recording is active. (Priority: **Must**)

- **REQ-F040**: The system shall set a video bitrate target of 2.5 Mbps for the MediaRecorder. (Priority: **Should**)

- **REQ-F041**: The system shall detect when the browser tab loses visibility (Page Visibility API) during recording and either warn the user or pause recording to prevent choppy output caused by background tab throttling. (Priority: **Must**)

- **REQ-F042**: The system shall impose a maximum recording duration limit (10-15 minutes) to prevent excessive memory consumption from Blob accumulation. (Priority: **Should**)

- **REQ-F043**: The system should offer MP4 (H.264) recording in Chrome 126+ where `MediaRecorder.isTypeSupported('video/mp4; codecs="avc1.42E01E, opus"')` returns true, providing a format directly compatible with social media platforms. (Priority: **Could**)

- **REQ-F044**: The system may offer optional WebM-to-MP4 conversion via ffmpeg.wasm for browsers that do not support direct MP4 recording, limited to clips under 5 minutes due to memory constraints. (Priority: **Could**)

#### Audio Capture

- **REQ-F045**: The system shall capture microphone audio via the `getUserMedia({ audio: true })` API. (Priority: **Must**)

- **REQ-F046**: The system shall route microphone input through a Web Audio API GainNode to allow user-adjustable volume control via a slider in the Vue UI. (Priority: **Must**)

- **REQ-F047**: The system shall display a visual audio level indicator (VU meter or peak indicator) so the user can confirm the microphone is active and receiving signal. (Priority: **Must**)

- **REQ-F048**: The system shall not play microphone audio back through the speakers to avoid echo/feedback — the audio signal is routed only to the recording pipeline. (Priority: **Must**)

- **REQ-F049**: The system shall gracefully handle microphone permission denial, allowing recording of video-only output without audio. (Priority: **Should**)

#### Scene Management

- **REQ-F050**: The system shall provide a library of pixel art background images that the user can select for a scene. (Priority: **Must**)

- **REQ-F051**: The system shall support three camera modes: Fixed (full stage view), Follow (tracks the active character), and Free Pan (manual camera position control). (Priority: **Must**)

- **REQ-F052**: The system shall allow the user to place static prop sprites in the scene via the UI, with props z-ordered by Y position (depth sorting). (Priority: **Should**)

- **REQ-F053**: Props shall support position, scale, layer assignment (foreground/background), and horizontal flip. (Priority: **Should**)

- **REQ-F054**: The system shall serialize the complete scene state (background, camera, characters, props) to a JSON format. (Priority: **Must**)

- **REQ-F055**: The system shall support saving scene state to localStorage and exporting as a JSON file. (Priority: **Must**)

- **REQ-F056**: The system shall support loading scene state from localStorage or an imported JSON file, restoring all scene elements to their saved positions and configurations. (Priority: **Must**)

- **REQ-F057**: The system shall support cloud save/load of scenes via Firebase Firestore for authenticated users. (Priority: **Could**)

- **REQ-F058**: The system shall support camera zoom control. (Priority: **Should**)

#### Multi-Character Support

- **REQ-F059**: The system shall support placing up to 4 characters in a single scene. (Priority: **Must**)

- **REQ-F060**: Each character shall be independently assignable to a specific gamepad (by gamepad index 0-3) via the Character Panel UI. (Priority: **Must**)

- **REQ-F061**: Multiple users shall be able to control different characters simultaneously, each using their own gamepad. (Priority: **Must**)

- **REQ-F062**: When a single user operates one gamepad, the system shall allow switching the active character assignment for that gamepad. (Priority: **Should**)

#### UI System

- **REQ-F063**: The UI shall be implemented as a Vue 3 overlay rendered on top of the Phaser canvas, with Vue and Phaser communicating exclusively through a typed EventBridge (mitt). (Priority: **Must**)

- **REQ-F064**: The UI shall include a Top Bar displaying scene name, recording status indicator, and elapsed recording timer. (Priority: **Must**)

- **REQ-F065**: The UI shall include a Character Panel for selecting characters, assigning gamepads, and choosing costumes. (Priority: **Must**)

- **REQ-F066**: The UI shall include a Scene Panel for selecting backgrounds and browsing/placing props. (Priority: **Must**)

- **REQ-F067**: The UI shall include Recording Controls (start, stop, countdown display). (Priority: **Must**)

- **REQ-F068**: The UI shall include a Preview Modal for video playback after recording, with download and discard options. (Priority: **Must**)

- **REQ-F069**: The UI shall include an Action Binding Editor for remapping gamepad buttons to actions. (Priority: **Should**)

- **REQ-F070**: The entire Vue UI overlay shall be hideable via the Select/Back gamepad button (REQ-F010) so recordings show only the game canvas. (Priority: **Must**)

- **REQ-F071**: Vue components shall never directly access Phaser game objects; Phaser scenes shall never directly manipulate Vue state. All cross-framework communication shall use the EventBridge. (Priority: **Must**)

#### Asset System

- **REQ-F072**: The system shall load character sprites as PNG sprite sheets paired with JSON animation definition files. (Priority: **Must**)

- **REQ-F073**: The system shall implement an asset manifest that defines available characters, backgrounds, and props, used for loading and caching. (Priority: **Must**)

- **REQ-F074**: Initial assets shall be sourced from open-license pixel art packs (itch.io, LPC). Attribution requirements for GPL/CC-BY-SA assets shall be respected and displayed in the application. (Priority: **Must**)

- **REQ-F075**: The MVP shall include at minimum: 1 character (knight) with full animation set covering all states (IDLE, WALK, RUN, JUMP, CROUCH, ACTION, EMOTE, DEAD), 1 background, and basic UI elements. (Priority: **Must**)

- **REQ-F076**: The full asset set shall include at least 3 character types, 5 backgrounds, and 15 props. (Priority: **Should**)

- **REQ-F077**: The system should support user-uploaded custom sprite sheets in future iterations. (Priority: **Won't** — deferred to post-v1)

#### Firebase / Cloud Features

- **REQ-F078**: The system shall support user authentication via Firebase Auth. (Priority: **Could**)

- **REQ-F079**: Authenticated users shall be able to save scenes to Firebase Firestore and load them from any device. (Priority: **Could**)

- **REQ-F080**: Authenticated users shall be able to upload recorded videos to Firebase Cloud Storage. (Priority: **Could**)

- **REQ-F081**: The system may provide shareable links for saved scenes and recordings. (Priority: **Could**)

#### Additional Features

- **REQ-F082**: The system shall provide speech bubbles that display above characters, triggered by user input or automatically during microphone activity. (Priority: **Could**)

- **REQ-F083**: The system shall support scene transitions (fade, cut, wipe) between different backgrounds or scene configurations. (Priority: **Could**)

- **REQ-F084**: The system shall provide a keyboard fallback for basic character control when no gamepad is connected. (Priority: **Should**)

- **REQ-F085**: The system shall include an onboarding flow or tutorial that introduces gamepad controls and the recording workflow to first-time users. (Priority: **Should**)

---

### Non-Functional Requirements

#### Performance

- **REQ-NF001**: The Phaser game loop shall render at a stable 60 frames per second on mid-range hardware (e.g., Intel i5 / Ryzen 5, integrated GPU, 8GB RAM) with up to 4 characters, 15 props, and 1 background active. (Priority: **Must**)

- **REQ-NF002**: The recording pipeline shall capture at 30 frames per second without causing the game loop to drop below 50fps. (Priority: **Must**)

- **REQ-NF003**: Gamepad input-to-visual-response latency shall not exceed 50ms (3 frames at 60fps), ensuring character movement feels immediate and responsive. (Priority: **Must**)

- **REQ-NF004**: Recording start (after countdown) shall begin capturing within 100ms of the final countdown tick. (Priority: **Should**)

- **REQ-NF005**: The total initial page load (including Phaser initialization and MVP asset loading) shall complete within 5 seconds on a 10 Mbps connection. (Priority: **Should**)

- **REQ-NF006**: Individual asset loading (single sprite sheet + JSON) shall complete within 500ms on a 10 Mbps connection. (Priority: **Should**)

- **REQ-NF007**: Memory usage during a 10-minute recording shall not exceed 500MB total (including Blob accumulation). (Priority: **Should**)

#### Browser Compatibility

- **REQ-NF008**: The application shall be fully functional on Chrome 120+ and Edge 120+ (Chromium-based). (Priority: **Must**)

- **REQ-NF009**: The application shall be functional on Firefox 120+ with the following documented limitations: VP8-only recording, potential audio track memory issues. (Priority: **Should**)

- **REQ-NF010**: The application shall not target Safari or mobile browsers. A clear "unsupported browser" message shall be displayed if accessed from Safari or a mobile device. (Priority: **Must**)

#### Usability

- **REQ-NF011**: A user familiar with gamepads shall be able to set up a scene and begin recording within 5 minutes of first use, without consulting external documentation. (Priority: **Should**)

- **REQ-NF012**: All gamepad button mappings shall be discoverable via on-screen reference (tooltip, overlay, or dedicated help panel). (Priority: **Should**)

- **REQ-NF013**: The application shall provide clear visual and textual feedback when no gamepad is detected, guiding the user to connect one. (Priority: **Must**)

#### Reliability

- **REQ-NF014**: If a gamepad disconnects during a recording, the system shall not crash. The affected character shall freeze in its last state, and a non-blocking notification shall inform the user. (Priority: **Must**)

- **REQ-NF015**: If the MediaRecorder encounters an error during recording, the system shall attempt to finalize and save any captured data, and display a clear error message. (Priority: **Must**)

- **REQ-NF016**: Scene save/load shall be idempotent — saving and loading a scene shall produce an identical scene state. (Priority: **Must**)

#### Maintainability

- **REQ-NF017**: The codebase shall be written in TypeScript with strict mode enabled, covering both Vue and Phaser code. (Priority: **Must**)

- **REQ-NF018**: The EventBridge shall use fully typed event definitions so that event payloads are checked at compile time. (Priority: **Must**)

- **REQ-NF019**: The Vue-Phaser boundary shall be enforced architecturally — no direct imports between `src/vue/` and `src/phaser/` directories. All communication through `src/bridge/` and `src/services/`. (Priority: **Must**)

#### Scalability

- **REQ-NF020**: The asset system shall support lazy loading — only assets required for the current scene are loaded, not the entire asset library. (Priority: **Should**)

- **REQ-NF021**: The architecture shall support adding new character types, backgrounds, and props without code changes — only new asset files and manifest entries. (Priority: **Should**)

---

## Use Cases

### UC-001: Record a Solo Scene

- **Actor**: Content Creator (single user)
- **Precondition**: User has the application open in Chrome/Edge. One gamepad is connected and detected. Microphone permission has been granted (or video-only mode accepted).
- **Main Flow**:
  1. User selects a background from the Scene Panel.
  2. User selects a character from the Character Panel; character appears on stage.
  3. System auto-assigns the connected gamepad (index 0) to the character.
  4. User moves the character with the left joystick and adjusts facing with the right joystick to test controls.
  5. User presses the Select button to hide the Vue UI overlay.
  6. User presses the Start button to begin recording.
  7. System displays a 3-2-1 countdown on the canvas, then begins recording (canvas + mic audio).
  8. Recording indicator (red dot + timer) appears on screen.
  9. User controls the character (movement, actions, emotes) and speaks dialogue into the microphone.
  10. User presses Start again to stop recording.
  11. System stops MediaRecorder and presents the Preview Modal with video playback.
  12. User watches the preview. If satisfied, clicks Download; if not, clicks Discard.
  13. If Download: browser saves the WebM file to the user's downloads folder.
- **Alternative Flows**:
  - **AF-1 (No microphone)**: At step 3 or on first recording attempt, if mic permission is denied, system offers video-only recording mode with a warning that audio will not be captured.
  - **AF-2 (Tab switch during recording)**: If the user switches browser tabs during step 9, the system detects visibility loss and either pauses recording or displays a persistent warning banner when the tab regains focus.
  - **AF-3 (Gamepad disconnects)**: If the gamepad disconnects during step 9, the character freezes, a notification appears, and recording continues. User can reconnect the gamepad or stop recording.
  - **AF-4 (Discard recording)**: At step 12, user clicks Discard. The preview modal closes and the recorded blob is released from memory. User returns to the live stage.
  - **AF-5 (Memory limit reached)**: During step 9, if the recording approaches the memory/duration limit (REQ-F042), the system displays a warning and auto-stops recording after the limit, proceeding to step 11.
- **Postcondition**: User has a downloaded WebM video file of the scene, or has discarded the recording and returned to the live stage.

---

### UC-002: Record a Multi-Character Scene

- **Actor**: Multiple Content Creators (2-4 users, each with a gamepad) or a single user managing multiple characters
- **Precondition**: Application is open. Multiple gamepads are connected and detected (2-4). Microphone permission granted.
- **Main Flow**:
  1. First user selects a background and places 2-4 characters on the stage via the Character Panel.
  2. For each character, a user assigns a gamepad via the Character Panel (dropdown showing gamepad indices 0-3).
  3. Each user tests their assigned character by moving their joystick; only their assigned character responds.
  4. One user presses Select to hide the UI.
  5. Any user presses Start on their gamepad to begin recording (all gamepads can trigger recording).
  6. System displays countdown, then begins recording.
  7. All users simultaneously control their characters — moving, performing actions, speaking dialogue. All voices captured through a single microphone (or multiple if audio hardware supports it).
  8. Any user presses Start to stop recording.
  9. Preview modal appears with the multi-character scene playback.
  10. User downloads or discards.
- **Alternative Flows**:
  - **AF-1 (Single user, multiple characters)**: At step 2, one user assigns their single gamepad to Character A. During performance (step 7), the user switches active character assignment to Character B (REQ-F062). Character A holds its last pose. User now controls Character B.
  - **AF-2 (Gamepad conflict)**: Two users attempt to assign the same gamepad index to different characters. System prevents this and displays a warning indicating the gamepad is already assigned.
  - **AF-3 (Mid-scene gamepad disconnect)**: One user's gamepad disconnects. Their character freezes. Other users continue. Recording continues.
- **Postcondition**: A downloaded video file showing multiple characters performing and interacting on a shared stage.

---

### UC-003: Set Up a Scene

- **Actor**: Content Creator
- **Precondition**: Application is open. At least one character type and one background are available in the asset library.
- **Main Flow**:
  1. User opens the Scene Panel.
  2. User browses the background library and selects a background (e.g., "castle-hall"). The stage canvas updates immediately.
  3. User opens the Character Panel and selects a character type (e.g., "knight"). A character instance appears on stage at a default position.
  4. User optionally adds additional characters (up to 4 total).
  5. User drags characters to desired starting positions on the stage (or uses gamepad to walk them into position).
  6. User browses the Props palette in the Scene Panel and clicks props to place them on stage.
  7. User adjusts prop position, scale, and layer (foreground/background) via UI controls.
  8. User selects a camera mode (Fixed, Follow, or Free Pan) from the Scene Panel.
  9. User adjusts camera zoom if needed.
  10. Scene is ready for recording (see UC-001 or UC-002).
- **Alternative Flows**:
  - **AF-1 (Change background mid-setup)**: At any point, user selects a different background. All characters and props remain in place; only the background image changes.
  - **AF-2 (Remove character/prop)**: User selects a character or prop and clicks a remove button. The element is removed from the stage.
  - **AF-3 (Costume change)**: User selects a character and uses D-pad Up (or UI) to cycle through available costumes for that character type.
- **Postcondition**: A fully configured scene with background, characters, and props positioned and ready for performance.

---

### UC-004: Remap Gamepad Controls

- **Actor**: Content Creator
- **Precondition**: Application is open. A gamepad is connected.
- **Main Flow**:
  1. User opens the Action Binding Editor from the UI.
  2. System displays the current button-to-action mapping in a list/grid format.
  3. User selects an action to remap (e.g., "Jump").
  4. System enters "listen mode" and prompts: "Press the button you want to assign to Jump."
  5. User presses a button on the gamepad.
  6. System detects the button press via the GamepadService, records the new mapping, and updates the display.
  7. If the newly pressed button was already assigned to another action, the system warns the user and either swaps the bindings or prompts for confirmation.
  8. User repeats for any other actions they wish to remap.
  9. User closes the Action Binding Editor. New mappings take effect immediately.
- **Alternative Flows**:
  - **AF-1 (Reset to defaults)**: User clicks a "Reset to Defaults" button. All bindings revert to the default mapping.
  - **AF-2 (No gamepad during remap)**: If no gamepad is detected when opening the editor, system displays a message asking the user to connect a gamepad before remapping.
  - **AF-3 (Cancel remap)**: User presses Escape or clicks Cancel during listen mode. The action retains its previous binding.
- **Postcondition**: The user's custom gamepad bindings are active and persisted (localStorage) for future sessions.

---

### UC-005: Save and Load a Scene

- **Actor**: Content Creator
- **Precondition**: Application is open with a scene configured (at least one character placed on a background).
- **Main Flow (Save)**:
  1. User clicks "Save Scene" in the UI.
  2. System serializes the current scene state to JSON (background, camera mode/zoom/position, all characters with positions/facing/costume/gamepad assignments, all props with positions/scale/layer/flip).
  3. System saves the JSON to localStorage under a named key.
  4. System confirms save with a brief toast notification.
- **Main Flow (Load)**:
  1. User clicks "Load Scene" in the UI.
  2. System displays a list of saved scenes (from localStorage).
  3. User selects a scene.
  4. System deserializes the JSON and restores all scene elements: background, character positions, prop placements, camera settings.
  5. Stage canvas updates to reflect the loaded scene.
- **Alternative Flows**:
  - **AF-1 (Export to file)**: User clicks "Export Scene" and the system downloads the scene JSON as a `.json` file.
  - **AF-2 (Import from file)**: User clicks "Import Scene" and selects a `.json` file from their filesystem. System validates the JSON structure and loads the scene.
  - **AF-3 (Invalid JSON on import)**: If the imported file has an invalid structure, the system displays an error message and does not modify the current scene.
  - **AF-4 (Overwrite existing save)**: User saves a scene with the same name as an existing save. System prompts for confirmation before overwriting.
  - **AF-5 (Cloud save — future)**: Authenticated user saves to Firestore (REQ-F057). Scene is accessible from any device after login.
  - **AF-6 (Missing assets on load)**: If the saved scene references a character type or background not present in the current asset manifest, the system displays a warning listing the missing assets and substitutes placeholders or skips the missing elements.
- **Postcondition**: Scene state is persisted (save) or fully restored from a previous save (load), with all elements matching the saved configuration.

---

### UC-006: Live Stream with Virtual Puppet

- **Actor**: Streamer
- **Precondition**: User has OBS or Streamlabs configured. Application is open in a browser. Gamepad connected. Microphone routed through OBS (not captured by the app).
- **Main Flow**:
  1. User sets up a scene in Virtual Stage (see UC-003).
  2. User hides the Vue UI overlay via the Select button for a clean canvas.
  3. In OBS, user adds the browser window or a browser source pointing to the application as a scene source.
  4. User begins their stream via OBS.
  5. User controls the pixel art character in real-time with the gamepad while speaking into their stream microphone.
  6. Viewers see the pixel art character moving and emoting in sync with the streamer's gamepad input.
  7. User does not use the in-app recording feature — OBS handles the stream and any recording.
- **Alternative Flows**:
  - **AF-1 (Transparent background)**: If a "transparent/chroma key" background option is available, user selects it so OBS can overlay the character on other content. [HUMAN-INPUT-NEEDED: Is chroma key / transparent background a planned feature?]
  - **AF-2 (Latency issues)**: If the streamer experiences input lag between gamepad and character movement visible on stream, they adjust Phaser render settings or OBS capture settings.
- **Postcondition**: The streamer has a live pixel art puppet responding to gamepad input, visible to their stream audience.

---

### UC-007: First-Time User Onboarding

- **Actor**: New user (any user type)
- **Precondition**: User has navigated to the application for the first time in their browser. No prior session data in localStorage.
- **Main Flow**:
  1. System detects first-time visit (no localStorage flag).
  2. System presents an onboarding overlay or guided tutorial.
  3. Tutorial walks the user through: connecting a gamepad, basic character movement (left stick), facing (right stick), performing actions (face buttons), using the emote wheel (bumper + face button), and starting a recording.
  4. User follows along, performing each action as prompted.
  5. Tutorial concludes with a prompt to set up their first scene.
  6. System sets a localStorage flag to avoid showing the tutorial on subsequent visits.
- **Alternative Flows**:
  - **AF-1 (Skip tutorial)**: User clicks "Skip" at any point. Tutorial closes. User proceeds to the main stage.
  - **AF-2 (Revisit tutorial)**: User accesses the tutorial again from a Help menu option.
  - **AF-3 (No gamepad on first visit)**: Tutorial detects no gamepad and focuses on scene setup, keyboard fallback, and general UI orientation instead.
- **Postcondition**: User understands the core workflow (control character, record scene, download video) and is ready to create content.

---

### UC-008: Adjust Audio Settings Before Recording

- **Actor**: Content Creator
- **Precondition**: Application is open. Microphone permission granted.
- **Main Flow**:
  1. User locates the microphone gain control slider in the UI.
  2. User speaks into the microphone while observing the visual audio level indicator (VU meter).
  3. User adjusts the gain slider until the audio level indicator shows their voice at an appropriate level (not clipping, not too quiet).
  4. User is satisfied with the audio level and proceeds to record (UC-001).
- **Alternative Flows**:
  - **AF-1 (Microphone not detected)**: System displays "No microphone detected" with guidance to check browser permissions or hardware connections.
  - **AF-2 (Switch microphone)**: If the user has multiple audio input devices, they use browser settings (or a future in-app device selector) to switch the active microphone.
- **Postcondition**: Microphone gain is set to an appropriate level for the recording session.

---

## Open Questions

1. **[HUMAN-INPUT-NEEDED] Monetization strategy**: The intake document states monetization is not yet defined. Premium asset packs, cloud storage tiers, and custom character commissions are mentioned as possibilities. What is the timeline for defining this? Does it affect MVP scope (e.g., should the asset system be designed for future paid asset delivery from day one)?

2. **[HUMAN-INPUT-NEEDED] Chroma key / transparent background**: Streamers using OBS would benefit from a transparent or green-screen background option so the puppet can be overlaid on other content. Is this a planned feature? If so, what priority?

3. **[HUMAN-INPUT-NEEDED] Phaser v3 vs v4**: Phaser 4.0.0 released April 10, 2026 (2 days before this analysis). The intake document specifies Phaser.js v3. The knowledge brief notes that v4 is the official recommendation but the ecosystem (rex plugins, Vue templates, Phavuer) is largely v3-targeted. Should the project start with v3 for ecosystem stability, or adopt v4 for long-term viability? This decision has significant architectural impact.

4. **[HUMAN-INPUT-NEEDED] MP4 output priority**: The primary recording format is WebM. Chrome 126+ supports direct MP4/H.264 recording. TikTok and Instagram prefer MP4 over WebM. Should MP4 direct recording (REQ-F043) be elevated from Could to Should? Should ffmpeg.wasm conversion (REQ-F044) be included in MVP?

5. **[HUMAN-INPUT-NEEDED] Maximum recording duration**: REQ-F042 suggests 10-15 minutes. Indie filmmakers (tertiary users) want 5-30 minute scenes. What is the hard limit? Is there a risk of alienating tertiary users with a 10-minute cap?

6. **[HUMAN-INPUT-NEEDED] Art asset licensing**: The project plans to use LPC sprites (GPL 3.0 / CC-BY-SA 3.0). GPL in particular may have implications for the application itself if sprites are considered "linked" with the code. Has legal review been performed on asset licensing compatibility?

7. **[HUMAN-INPUT-NEEDED] Keyboard fallback scope**: REQ-F084 mentions keyboard fallback for basic character control. What is the expected scope — full parity with gamepad controls, or a limited subset (movement + a few actions)? This significantly affects development effort.

8. **[HUMAN-INPUT-NEEDED] OBS integration**: For the streaming use case (UC-006), is there any plan for native OBS integration (e.g., OBS Browser Source API, NDI output), or is it purely "capture the browser window"?

9. **[HUMAN-INPUT-NEEDED] Multi-microphone support**: In a multi-user recording session (UC-002), the project context mentions "All voices captured through a single microphone." Should the system support multiple microphone inputs (one per participant), or is a single shared microphone the design intent?

10. **[HUMAN-INPUT-NEEDED] Accessibility considerations**: No accessibility requirements are mentioned. Should the tool support any accessibility features (colorblind modes, screen reader support for UI elements, alternative input methods)?

---

## Assumptions

1. **A-001**: The project will use Phaser v3 (3.90.0) as specified in the intake document, not the newly released Phaser v4, unless the Architect recommends otherwise and the product owner approves. *Rationale*: The intake explicitly lists Phaser.js v3, and the v3 ecosystem (plugins, templates, community resources) is more mature.

2. **A-002**: All users have access to at least one standard-layout gamepad (Xbox, PlayStation, or Switch Pro controller). The application is not designed to function fully without a gamepad, though a keyboard fallback provides basic capability. *Rationale*: The entire product concept centers on gamepad-as-puppet-controller. Keyboard is an accessibility concession, not a primary input.

3. **A-003**: Users will not switch browser tabs during recording. The system will warn about this but cannot prevent it. Background tab throttling causing choppy recordings is an accepted limitation with mitigation (Page Visibility API detection). *Rationale*: This is a browser-level constraint that cannot be worked around without OS-level capture APIs.

4. **A-004**: The MVP (Phase 3 completion) requires only 1 character, 1 background, and WebM recording. Multi-character, props, scene save/load, and cloud features are post-MVP. *Rationale*: Explicit in the development phases outlined in the project context.

5. **A-005**: Audio capture is single-microphone only. In multi-user sessions, all participants share one microphone or use an external mixer. The application does not mix multiple microphone inputs. *Rationale*: The project context describes a single getUserMedia audio stream feeding into the recording pipeline.

6. **A-006**: The application is a standalone browser application with no server-side component for MVP. Firebase services (Auth, Firestore, Cloud Storage) are Phase 6 additions. Local-only operation is the default. *Rationale*: The development phases explicitly defer Firebase to Phase 6.

7. **A-007**: Video output quality at 2.5 Mbps VP9/Opus in WebM is acceptable for the target content formats (YouTube, TikTok shorts). Users who need higher quality can use OBS for screen capture. *Rationale*: 2.5 Mbps at 720p/30fps produces good quality for pixel art content, which has low visual complexity (large flat-color areas compress well).

8. **A-008**: The deadzone threshold of 0.15 is appropriate as a default for most controllers. It falls within the "typical" range (0.05-0.12) to "noticeable" range (0.13-0.25) boundary identified in the tech brief, providing reasonable drift prevention without sacrificing too much precision. *Rationale*: The project context specifies 0.15; the tech brief confirms this is within standard industry range.

9. **A-009**: 4-directional sprites with horizontal flip is sufficient for the character animation system. 8-directional sprites are not required for MVP. *Rationale*: The project context specifies "4-directional sprites (down, up, left, right) with horizontal flip for left/right." The LPC sprite system also uses 4 directions.

10. **A-010**: The "theatrical actions" design (no game mechanics, no hitboxes) means character-to-character interaction is purely visual. Characters can overlap and pass through each other. There is no collision detection between characters. *Rationale*: The project context explicitly states "All actions are theatrical/expressive — no game mechanics, no hitboxes, no damage."

11. **A-011**: The recording countdown (3-2-1) is displayed as a visual overlay on the canvas itself (not just in the Vue UI), so it appears in the recorded output as a visual cue — or it is rendered before recording begins so the countdown is not captured. *Rationale*: This needs clarification but the most intuitive behavior is that the countdown occurs before recording begins, so the final video starts cleanly. Assumed: countdown happens, then recording starts.

12. **A-012**: The Select/Back button hides all Vue UI elements but does not hide any Phaser-rendered UI elements (e.g., recording indicator on the canvas). If the recording indicator is rendered on the canvas, it will appear in the recorded video. *Rationale*: The recording indicator location (Vue overlay vs. Phaser canvas) determines whether it appears in recordings. Assumed: the recording indicator is rendered in the Vue overlay and is therefore hideable, resulting in a clean recording.

---

## Traceability Matrix

| Requirement | Use Case(s) | Phase |
|-------------|-------------|-------|
| REQ-F001 to REQ-F007 | UC-001, UC-002, UC-004 | Phase 1 |
| REQ-F008 | UC-001, UC-002 | Phase 2 |
| REQ-F009, REQ-F010 | UC-001, UC-002, UC-006 | Phase 3 |
| REQ-F011 to REQ-F016 | UC-003, UC-004 | Phase 2, 5 |
| REQ-F017 | UC-001, UC-002, UC-004 | Phase 1 |
| REQ-F018 to REQ-F031 | UC-001, UC-002, UC-003 | Phase 1, 2 |
| REQ-F032 to REQ-F044 | UC-001, UC-002 | Phase 3 |
| REQ-F045 to REQ-F049 | UC-001, UC-008 | Phase 2, 3 |
| REQ-F050 to REQ-F058 | UC-003, UC-005 | Phase 4 |
| REQ-F059 to REQ-F062 | UC-002 | Phase 4 |
| REQ-F063 to REQ-F071 | UC-001 to UC-008 | Phase 1-5 |
| REQ-F072 to REQ-F077 | UC-003 | Phase 1, 4, 5 |
| REQ-F078 to REQ-F081 | UC-005 (AF-5) | Phase 6 |
| REQ-F082 to REQ-F085 | UC-003, UC-006, UC-007 | Phase 5 |
| REQ-NF001 to REQ-NF007 | All | Phase 1-3 (validated continuously) |
| REQ-NF008 to REQ-NF010 | All | Phase 1 (validated continuously) |
| REQ-NF011 to REQ-NF013 | UC-007 | Phase 5 |
| REQ-NF014 to REQ-NF016 | UC-001, UC-002, UC-005 | Phase 3, 4 |
| REQ-NF017 to REQ-NF019 | All (architectural) | Phase 1 |
| REQ-NF020 to REQ-NF021 | UC-003 | Phase 4, 5 |

---

## Risk-Requirement Mapping

| Risk | Related Requirements | Mitigation |
|------|---------------------|------------|
| Gamepad API browser inconsistency | REQ-F001, REQ-F002, REQ-F005 | Runtime detection, configurable deadzone (REQ-F016), remappable bindings (REQ-F015), Standard Gamepad mapping only |
| MediaRecorder codec fragmentation | REQ-F034, REQ-F035 | Runtime codec detection via `isTypeSupported()`, fallback chain (VP9 -> VP8), optional MP4 path (REQ-F043) |
| Background tab throttling corrupts recordings | REQ-F041 | Page Visibility API detection, user warning, pause-on-hide option |
| Recording memory exhaustion on long sessions | REQ-F042, REQ-NF007 | Duration cap (10-15 min), memory monitoring, user warnings |
| Open-license asset availability | REQ-F074, REQ-F075, REQ-F076 | LPC sprite system provides baseline, CC0 assets on itch.io, plan for commissioned original art |
| Vue-Phaser lifecycle conflicts | REQ-F063, REQ-F071, REQ-NF019 | Strict EventBridge boundary, no cross-framework state sharing, typed events |
| Firefox audio track memory leak | REQ-NF009 | Document as known Firefox limitation, recommend Chrome/Edge for recording |

---

*End of Business Analysis*
