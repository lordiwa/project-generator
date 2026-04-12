# Expert Analysis: Product & UX — Virtual Stage

**Agent**: Product/UX Expert
**Project**: Virtual Stage
**Date**: 2026-04-12
**Input Sources**: intake.md, expert-research-analyst.md, expert-business-analyst.md, pixel-art-machinima-puppeteering-brief.md

---

## User Personas

### The Skit Creator

- **Role**: Short-form content creator producing comedy and drama clips (30s-3min) for TikTok, YouTube Shorts, and Instagram Reels. Typically 16-30 years old with a small but growing audience. May also create memes, reaction clips, or story-driven series in episodic format.
- **Goals**: Produce entertaining pixel art skits quickly without learning animation software. Go from idea to finished video in under 30 minutes. Output directly in a format TikTok/YouTube accepts (MP4 preferred). Build a recognizable style with recurring characters. Maintain a frequent posting cadence (3-5 clips per week).
- **Pain Points**: Existing animation tools (Aseprite, After Effects) require frame-by-frame work that takes hours per second of output. Source Filmmaker and Garry's Mod are 3D and have steep learning curves. VTuber tools only do expression switching, not full character movement. Needs to convert WebM to MP4 before uploading to most platforms. Has a gamepad sitting on their desk but no creative tool uses it.
- **Context**: Works from a single-monitor desktop or laptop. Has a USB gamepad (Xbox or PS5 controller). Uses a basic USB microphone or headset mic. Works in short bursts (15-45 minutes per session). Browses TikTok and YouTube for inspiration, then immediately wants to create. Values speed over production polish.

### The Stream Puppeteer

- **Role**: Twitch or YouTube Live streamer who wants a pixel art puppet reacting in real-time during streams. Ranges from hobbyist streamers (50-500 viewers) to mid-tier creators. May already use veadotube mini or a PNGTuber setup for expression switching and wants something more expressive.
- **Goals**: Control a pixel art character with a gamepad while speaking to the audience, so the character walks, emotes, and acts out what the streamer is saying. Use the tool as a scene source in OBS with a clean (UI-free) canvas. Switch between characters or costumes during a stream. Maintain low-latency response so character movement feels live.
- **Pain Points**: veadotube mini only switches between static expressions based on mic input — the character cannot walk, jump, or perform actions. No existing tool lets them puppeteer a pixel art character spatially during a live stream. Current PNGTuber setups feel static and limit creative expression. Needs to manage OBS, the game/content they are streaming, and the puppet tool simultaneously — cognitive load is high.
- **Context**: Dual-monitor setup. OBS running with multiple scene sources. One gamepad dedicated to character control (separate from any game they might be playing). Stream sessions last 2-6 hours. Does not use the in-app recording feature — OBS handles capture. Needs the character canvas to run stably for extended periods without memory leaks or frame drops. Values reliability and low latency over feature richness.

### The Indie Filmmaker

- **Role**: Animation hobbyist or indie creator producing longer narrative pixel art shorts (3-15 minutes). May be a solo creator or collaborating with 1-2 friends doing voice acting. Often has a game dev or pixel art background. Produces content for YouTube or film festivals/jams.
- **Goals**: Direct multi-character scenes with precise staging. Record multiple takes and keep the best one. Save and reload scene configurations between sessions. Achieve consistent visual quality across a multi-scene project. Control camera framing (zoom, pan, follow) for cinematic effect. Produce content that stands out from typical short-form clips.
- **Pain Points**: No tool lets them direct pixel art scenes in real-time — everything is either frame-by-frame (Aseprite), requires rigging knowledge (Spine, Inochi2D), or is 3D (iClone, SFM). Coordinating multi-character scenes where each character needs distinct movement is extremely difficult in traditional animation. Recording sessions for longer pieces need save/load so they can work across multiple days. Browser-based recording has duration limits that may cut off longer scenes.
- **Context**: Dedicated workspace with a good microphone setup. May use multiple gamepads (solo, switching between characters, or with voice actor friends each holding a controller). Works in longer sessions (1-3 hours). Plans scenes in advance with notes or storyboards. Expects to iterate — record, review, adjust, re-record. Values scene management, camera control, and output quality.

### The Party Host

- **Role**: Social creator who gets 2-4 friends together (locally, same room) for collaborative improv pixel art sessions. Treats Virtual Stage like a party game or group creative activity. May or may not post the resulting videos — the fun is in the collaborative performance itself.
- **Goals**: Get everyone set up and controlling characters as fast as possible with minimal friction. Have all 4 gamepads working simultaneously. Capture the chaos of collaborative improv as a single video. Share the resulting clip with the group (and possibly post it). Minimize setup time so the group stays engaged.
- **Pain Points**: Getting 4 gamepads recognized and assigned to 4 characters is a potential friction point. If setup takes more than 5 minutes, the group loses interest. No existing tool supports multi-gamepad collaborative content creation in a browser. Single-microphone capture means everyone needs to be near the mic, or the audio is uneven. Explaining controls to friends who have never used the tool before requires an at-a-glance reference.
- **Context**: Living room or shared space. 4 controllers (possibly mixed — Xbox, PS5, Switch Pro). One laptop or desktop connected to a TV or large monitor. One shared microphone (laptop mic or a single USB mic placed centrally). Short sessions (15-45 minutes). High energy, low patience for configuration. Needs a "quick start" path that bypasses detailed setup.

---

## User Journeys

### Journey 1: First-Time Experience

**Persona**: Any new user (most commonly The Skit Creator)

1. **Land on the tool** -> User opens the URL in Chrome/Edge -> Browser loads the application shell (Vue UI + Phaser canvas). The canvas shows a default background with no characters placed. The UI panels are visible in their default (collapsed or sidebar) state.
2. **First-visit detection** -> System checks localStorage for a session flag -> No flag found, so the system presents an onboarding overlay on top of the canvas.
3. **Gamepad detection gate** -> Onboarding checks `navigator.getGamepads()` -> If a gamepad is connected, the overlay shows: "Gamepad detected! Let's get you started." If no gamepad is detected, the overlay shows: "Connect a gamepad to control characters. Press any button to activate it." with a visual diagram of a standard controller. A small "Use keyboard instead" link is available as a fallback.
4. **Step 1 — Meet the stage** -> Onboarding highlights the canvas area and explains: "This is your stage. Pixel art characters perform here." A pre-placed character appears on a sample background.
5. **Step 2 — Move your character** -> Onboarding highlights the left joystick on a gamepad diagram: "Move the left stick to walk. Push further to run." User moves the joystick and sees the character respond immediately. The onboarding waits for the user to move the character before proceeding (or offers a "Next" button to skip).
6. **Step 3 — Look and emote** -> Onboarding highlights the right joystick and face buttons: "Right stick controls where the character faces. Press A/B/X/Y for actions." User experiments. Onboarding shows a brief button map overlay on the canvas (semi-transparent, showing the standard gamepad layout with action labels).
7. **Step 4 — Record your first scene** -> Onboarding explains: "Press Start on your gamepad (or click Record) to capture a video. Try it now!" User presses Start. A 3-2-1 countdown appears. Recording begins. User performs for 5-10 seconds. Onboarding prompts: "Press Start again to stop." User stops. Preview modal appears with the recorded clip.
8. **Step 5 — Download** -> Preview modal shows the clip with Play, Download, and Discard buttons. Onboarding highlights Download: "Save your clip and share it!" User downloads. Onboarding concludes with: "You're ready. Set up your own scenes from the panels on the left." and a link to revisit the tutorial from the Help menu.
9. **Session flag set** -> System writes a localStorage flag so the onboarding does not appear again.

- **Success Criteria**: User completes onboarding in under 3 minutes. User has moved a character, performed at least one action, and downloaded a recording before the onboarding ends. User understands the relationship between gamepad input, character movement, and recorded output.
- **Drop-off Risks**: 
  - No gamepad connected and the keyboard fallback is not obvious -> user leaves immediately. Mitigation: prominent "Use keyboard" link in the no-gamepad state with clear WASD instructions.
  - Onboarding is too long or feels like a mandatory tutorial -> user clicks away. Mitigation: every step has a "Skip" option; the entire onboarding can be dismissed with one click.
  - The first recording produces a WebM file that the user cannot open or upload -> confusion. Mitigation: record to MP4 on Chrome/Edge (direct H.264 support); display a clear format label on the download button.

### Journey 2: Solo Recording Session

**Persona**: The Skit Creator or The Indie Filmmaker

1. **Open the tool** -> User loads the application. Previous session state may be restored from localStorage, or user starts fresh.
2. **Set up scene** -> User opens the Scene Panel and selects a background. Canvas updates immediately. User opens the Character Panel and places a character on stage (click to add, character appears at center stage).
3. **Position character** -> User either drags the character to a starting position via mouse on the canvas, or picks up the gamepad and walks the character into position using the left stick.
4. **Optional: Add props** -> User browses the Props section of the Scene Panel, clicks props to place them on stage, and adjusts position/scale with drag handles.
5. **Optional: Set camera** -> User selects a camera mode (Fixed, Follow, Free Pan) and adjusts zoom. For a solo skit, "Fixed" at a comfortable zoom level is most common.
6. **Audio check** -> User glances at the microphone level indicator in the top bar or recording controls area. If active, a small VU meter shows the current mic input level. User adjusts gain if needed via a slider. If no mic is detected, a muted-mic icon with a tooltip explains how to grant permission.
7. **Hide UI** -> User presses Select/Back on the gamepad (or clicks a "Hide UI" toggle). All Vue panels and controls fade out, leaving only the clean Phaser canvas. A small, semi-transparent "REC" badge remains visible in a corner (rendered in the Vue layer, so it will not appear in the recording).
8. **Start recording** -> User presses Start on the gamepad. A 3-2-1 countdown appears on the canvas (rendered by Phaser, visible to the user but triggered before the MediaRecorder starts, so the countdown does not appear in the final video). After "1", recording begins. The recording indicator (red dot + elapsed timer) appears in the Vue overlay (hidden from recording).
9. **Perform** -> User controls the character with the gamepad while speaking dialogue into the microphone. The character walks, emotes, attacks, crouches, and reacts as directed by the user's gamepad input.
10. **Stop recording** -> User presses Start again. Recording stops. The Vue UI fades back in. The Preview Modal appears with the recorded video.
11. **Preview** -> User watches the clip in the Preview Modal. Controls: Play/Pause, scrub bar, volume. Below the video: "Download" (primary action, large button) and "Discard" (secondary action, text link or smaller button).
12. **Download or retry** -> If satisfied, user clicks Download. The file saves to their downloads folder as an MP4 (Chrome/Edge) or WebM (Firefox). If not satisfied, user clicks Discard, returns to the live stage, and can immediately start another take.

- **Success Criteria**: User goes from opening the tool to a downloaded video in under 10 minutes (including scene setup). The recording workflow (Start -> perform -> Start -> preview -> download) becomes muscle memory after 2-3 sessions.
- **Drop-off Risks**: 
  - Tab switch during recording ruins the take with no warning -> user wastes time. Mitigation: Page Visibility API detection with an immediate, prominent warning banner and optional auto-pause.
  - Preview modal video does not play (codec issue) -> user cannot evaluate the take. Mitigation: use a `<video>` element with the same Blob URL; test playback before showing the modal.
  - User forgets to hide UI before recording and the panels appear in the video -> user has to re-record. Mitigation: when recording starts, auto-hide the Vue UI if it is visible (with a brief "UI hidden for recording" toast).

### Journey 3: Multi-Player Session

**Persona**: The Party Host (or The Indie Filmmaker with voice actor friends)

1. **Host sets up scene** -> One person (the host) opens the tool and sets up the background, places 2-4 characters on stage, and optionally adds props. This is done via mouse/UI before gamepads are involved.
2. **Connect gamepads** -> Each participant connects their gamepad. The Gamepad Status indicator in the UI updates in real-time: "Gamepad 1: Xbox Controller", "Gamepad 2: DualSense", etc. As each gamepad is connected, it appears in the list. The system requires a button press on each gamepad to activate it (browser Gamepad API requirement).
3. **Assign gamepads to characters** -> The Character Panel shows each placed character with a dropdown for gamepad assignment. The host (or each participant) selects which gamepad controls which character. To assist identification, the system highlights the character when a gamepad is selected in the dropdown, and shows a "Press any button to identify" prompt — when a button is pressed on a gamepad, the corresponding dropdown flashes or highlights, so participants know which gamepad index corresponds to their physical controller.
4. **Quick test** -> Each participant moves their joystick to verify they control the correct character. Characters move independently. If assignment is wrong, the host swaps assignments in the Character Panel.
5. **Microphone setup** -> A single shared microphone captures all participants. The host checks the mic level indicator and adjusts gain. All participants should be roughly equidistant from the mic.
6. **Hide UI and record** -> The host presses Select to hide UI, then Start to begin recording. The 3-2-1 countdown signals everyone to get ready.
7. **Collaborative performance** -> All participants control their characters simultaneously, speaking dialogue, performing actions, and improvising. The single microphone captures all voices.
8. **Stop and review** -> Any participant presses Start to stop. Preview modal appears. The group watches together. Download or discard.

- **Success Criteria**: All 4 gamepads are recognized, assigned, and controlling independent characters within 3 minutes of connecting. The group recording workflow adds no more than 2 minutes of overhead compared to a solo session.
- **Drop-off Risks**: 
  - Gamepad identification confusion (which physical controller is "Gamepad 2"?) -> assignments are wrong and characters respond to the wrong person. Mitigation: the "press any button to identify" interaction in the assignment UI.
  - One participant's gamepad disconnects mid-recording -> their character freezes and the take may be ruined. Mitigation: non-blocking notification, character freezes gracefully, recording continues so other participants' performances are not lost.
  - The group cannot figure out controls without individual onboarding -> fun stalls. Mitigation: a printable/shareable "Quick Controls" reference card accessible from the Help menu (also displayable as an overlay).

### Journey 4: Scene Setup (Detailed)

**Persona**: The Indie Filmmaker (most thorough scene setup) or The Skit Creator (quick setup)

1. **Open Scene Panel** -> User clicks the Scene tab in the left sidebar (or the Scene Panel is visible by default). The panel shows two sections: "Backgrounds" and "Props."
2. **Choose background** -> The Backgrounds section displays a scrollable grid of thumbnail previews. User clicks a thumbnail. The Phaser canvas updates instantly to show the selected background. A subtle loading indicator appears if the background asset needs to be fetched.
3. **Place characters** -> User switches to the Character Panel (adjacent tab in the sidebar). The panel shows available character types (e.g., Knight, Mage, Rogue) as cards with sprite preview thumbnails. User clicks a character type to add an instance to the stage. The character appears at the center of the visible canvas area.
4. **Position characters** -> User can drag characters on the canvas (mouse interaction) or use the gamepad to walk them into position. For precise placement, a position coordinate readout appears next to the character when selected. Clicking a character on the canvas selects it in the Character Panel and vice versa.
5. **Assign costumes** -> With a character selected, the Character Panel shows available costumes/variants for that character type as a horizontal scrollable strip. User clicks to change costume. Alternatively, D-pad Up on the assigned gamepad cycles costumes.
6. **Add props** -> User opens the Props section of the Scene Panel. Props are displayed as a categorized grid (e.g., Furniture, Nature, Items, Decorations). User clicks a prop to add it to the stage at the center of the canvas. User drags the prop to the desired position.
7. **Adjust props** -> When a prop is selected (clicked on canvas or in the Props list), a small floating toolbar appears near the prop with controls: resize handles (corner drag), flip horizontal toggle, and a layer toggle (foreground/background). Props placed in the "background" layer render behind characters; "foreground" props render in front.
8. **Set camera** -> The Camera section at the bottom of the Scene Panel offers three mode buttons: Fixed (full stage), Follow (tracks active character), Free Pan (drag to pan). A zoom slider sits below the mode buttons. User selects a mode and adjusts zoom.
9. **Save scene** -> User clicks "Save" in the Scene Panel header. A save dialog appears with a name field (pre-filled with the current scene name or "Untitled Scene"). User names the scene and clicks Save. A toast notification confirms: "Scene saved." The scene JSON is stored in localStorage.
10. **Begin recording** -> Scene is fully configured. User proceeds to the recording workflow (Journey 2, step 7 onward).

- **Success Criteria**: A user can set up a complete scene (background + 2 characters + 3 props + camera) in under 5 minutes. Scene save/load is reliable and deterministic (loading a saved scene produces an identical state).
- **Drop-off Risks**: 
  - Too many clicks to place and adjust a prop -> scene setup feels tedious. Mitigation: drag-and-drop from the panel directly onto the canvas; sensible defaults for scale and layer.
  - User places characters but cannot figure out how to assign gamepads -> the scene is set up but unplayable. Mitigation: when a gamepad input is detected and no character is assigned to it, show a prompt: "Gamepad detected. Assign it to a character in the Character Panel."
  - User builds an elaborate scene and the browser crashes, losing everything -> devastating experience. Mitigation: auto-save to localStorage every 30 seconds with a recoverable "last session" restore on next load.

---

## Feature Prioritization

### P0 — Must Have for MVP (Phase 1-3)

| Feature | User Value | Effort | Priority | Rationale |
|---------|-----------|--------|----------|-----------|
| Gamepad character movement (left stick walk/run) | High | Medium | P0 | Core product interaction — without this, the tool has no purpose |
| Gamepad look-at direction (right stick) | High | Low | P0 | Essential for expressive character control; puppet feel requires independent facing |
| Face button actions (jump, attack, magic, nod) | High | Medium | P0 | Minimum viable expressiveness for character performance |
| Character state machine (IDLE, WALK, RUN, JUMP, ACTION, EMOTE, DEAD) | High | Medium | P0 | Required for coherent character animation; prevents animation glitches |
| Sprite sheet rendering with JSON animation definitions | High | Medium | P0 | Foundation of the visual system; characters cannot appear without this |
| Single background selection | High | Low | P0 | Minimum scene composition — a character needs a stage to perform on |
| Canvas video recording (captureStream + MediaRecorder) | High | High | P0 | Core value proposition — the tool produces video content |
| Microphone audio capture and mixing into recording | High | Medium | P0 | Voice acting is half the product; video without audio is incomplete |
| Recording start/stop via gamepad (Start button) | High | Low | P0 | Hands-on-controller workflow — user should not need to reach for the mouse mid-take |
| 3-2-1 countdown before recording | Medium | Low | P0 | Prevents awkward starts; gives the performer a moment to prepare |
| Recording preview modal (play, download, discard) | High | Medium | P0 | User must evaluate the take before committing to save or redo |
| Video file download (MP4 on Chrome/Edge, WebM fallback) | High | Medium | P0 | The deliverable — without download, the recording has no value |
| Vue UI overlay on Phaser canvas with EventBridge | High | High | P0 | Architectural foundation; all UI depends on this |
| UI hide/show via Select button | High | Low | P0 | Clean recordings require hiding UI panels; critical for output quality |
| Recording indicator (red dot + timer) in Vue overlay | Medium | Low | P0 | User must know recording is active; placed in Vue layer so it does not appear in video |
| Mic level indicator (VU meter) | Medium | Low | P0 | User needs to confirm mic is working before recording a take |
| No-gamepad detection and guidance message | High | Low | P0 | Without this, users with no gamepad see a dead screen with no guidance |
| Tab visibility detection during recording (Page Visibility API) | High | Low | P0 | Prevents silently ruined recordings — the highest-risk UX pitfall identified |
| Radial deadzone filtering on joysticks | Medium | Low | P0 | Without this, characters drift on their own — breaks the puppet illusion |

### P1 — Should Have (Phase 4-5)

| Feature | User Value | Effort | Priority | Rationale |
|---------|-----------|--------|----------|-----------|
| Emote wheel (LB/RB + face buttons = 8 emotes) | High | Medium | P1 | Dramatically expands expressiveness; differentiates from basic gamepad control |
| Multi-gamepad support (up to 4 controllers) | High | Medium | P1 | Enables the collaborative/party use case; strongest viral differentiator |
| Gamepad-to-character assignment UI | High | Medium | P1 | Required for multi-gamepad; must be intuitive with gamepad identification |
| Multiple character types (3+) | High | Medium | P1 | Visual variety; enables cast-based storytelling |
| Multiple backgrounds (5+) | Medium | Low | P1 | Scene variety for different skit settings |
| Prop placement system | Medium | High | P1 | Adds scene composition depth; important for filmmakers |
| Camera modes (Fixed, Follow, Free Pan) | Medium | Medium | P1 | Cinematic control; Follow mode is essential for single-character focused recording |
| Camera zoom | Medium | Low | P1 | Framing control; close-ups for dramatic moments |
| Scene save to localStorage | High | Medium | P1 | Session persistence; prevents work loss; enables multi-session projects |
| Scene load from localStorage | High | Medium | P1 | Complementary to save; enables returning to previous work |
| Scene export/import as JSON file | Medium | Low | P1 | Enables scene sharing between users; backup mechanism |
| Keyboard fallback for basic controls | Medium | Medium | P1 | Accessibility and "no gamepad" use case; broadens potential user base |
| Onboarding tutorial for first-time users | High | Medium | P1 | Critical for retention; users who do not understand controls in 2 minutes will leave |
| Costume cycling (D-pad Up) | Low | Low | P1 | Minor but adds variety; easy to implement once multi-costume assets exist |
| Theatrical death + "get up" (D-pad Down) | Low | Low | P1 | Comedic value for skits; low effort if death animation exists in the sprite set |
| Run lock toggle (L3) | Low | Low | P1 | Quality-of-life for extended movement without holding the stick at full tilt |
| Gamepad button remapping (Action Binding Editor) | Medium | Medium | P1 | Accommodates different controller preferences and accessibility needs |
| Crouch (LT) and Block (RT) holds | Medium | Low | P1 | Expands action vocabulary; useful for combat skits |
| Auto-hide UI when recording starts | Medium | Low | P1 | Prevents the mistake of recording with panels visible |
| Max recording duration cap with warning | Medium | Low | P1 | Prevents memory exhaustion; protects long sessions |

### P2 — Nice to Have (Phase 5-6 and Beyond)

| Feature | User Value | Effort | Priority | Rationale |
|---------|-----------|--------|----------|-----------|
| Speech bubbles (text overlays on characters) | Medium | Medium | P2 | Enhances storytelling but adds UI complexity; not core to the puppet paradigm |
| Scene transitions (fade, cut, wipe) | Medium | High | P2 | Cinematic polish; high effort for a browser tool; defer until core is solid |
| Firebase authentication | Low | Medium | P2 | Enables cloud features but MVP works entirely locally |
| Cloud scene save/load (Firestore) | Medium | Medium | P2 | Cross-device access; requires auth infrastructure first |
| Cloud recording storage | Medium | High | P2 | Convenience feature; users can use local download in the meantime |
| Shareable scene/recording links | Medium | Medium | P2 | Viral growth mechanism but depends on cloud infrastructure |
| ffmpeg.wasm WebM-to-MP4 conversion (Firefox fallback) | Low | High | P2 | Small user segment (Firefox); high bundle size cost; defer until Firefox usage warrants it |
| Chroma key / transparent background for OBS | High (Streamers) | Medium | P2 | Very valuable for the streaming persona but a niche use case overall |
| User-uploaded custom sprite sheets | High (Filmmakers) | High | P2 | Dramatically expands creative potential but requires validation, format handling |
| Configurable deadzone per gamepad | Low | Low | P2 | Power user feature; default deadzone is sufficient for most controllers |
| Multiple microphone input support | Low | High | P2 | Niche multi-user scenario; external mixer is a reasonable workaround |
| Aseprite JSON direct import | Medium | Medium | P2 | Streamlines the artist-to-stage pipeline; valuable once custom uploads are supported |
| "Performed live" metadata in exported video | Low | Low | P2 | Branding/authenticity signal; low effort but low immediate impact |

---

## Information Architecture

### Overall Layout Model

The application uses a **split layout**: a Vue 3 UI shell wrapping a Phaser.js game canvas. The Vue layer handles all panels, controls, menus, and overlays. The Phaser layer handles the game world (characters, backgrounds, props, camera). The two communicate exclusively via a typed EventBridge (mitt).

```
+----------------------------------------------------------------------+
|  TOP BAR                                                              |
|  [Scene Name]        [Mic Level ||||]  [REC *] 00:00:00   [?] [gear] |
+------+---------------------------------------------------------------+
|      |                                                                |
|  S   |                                                                |
|  I   |                     PHASER CANVAS                              |
|  D   |                   (Game World)                                 |
|  E   |                                                                |
|  B   |              Characters, Background, Props                     |
|  A   |                   Camera viewport                              |
|  R   |                                                                |
|      |                                                                |
|      +----------------------------------------------------------------+
|      |  BOTTOM TRAY (collapsible)                                     |
|      |  [Record] [Pause] [Stop]  |  Camera: [Fixed|Follow|Pan] Zoom  |
+------+---------------------------------------------------------------+
```

### Top Bar (always visible, auto-hides during recording)

- **Left**: Scene name (editable inline click-to-rename), scene saved/unsaved indicator (dot)
- **Center**: Microphone level indicator (VU meter bar), microphone gain quick-adjust (click the VU meter to expand a gain slider)
- **Right**: Recording status (idle: gray circle, recording: pulsing red dot + elapsed time "02:34"), Help button (? icon), Settings gear icon

### Left Sidebar (collapsible, tabbed)

The sidebar uses a vertical tab strip. Each tab opens a panel. Only one panel is open at a time.

**Tab 1: Characters**
- Character type grid (visual cards with sprite preview, name)
- Click card to add that character to the stage
- Below the grid: list of placed characters with:
  - Character name/type icon
  - Gamepad assignment dropdown (Gamepad 1 / Gamepad 2 / ... / None)
  - "Identify" button (press a button on any gamepad to highlight which index it is)
  - Costume selector (horizontal thumbnail strip)
  - Remove button (trash icon)

**Tab 2: Scenes**
- Backgrounds section: scrollable grid of background thumbnails with labels
- Click to set the active background
- Props section (below backgrounds): categorized grid of prop thumbnails
  - Category filter buttons (All, Furniture, Nature, Items, Decor)
  - Click prop to add to stage at center of canvas
  - Placed props list (below the grid): name, position readout, layer toggle (FG/BG), remove button

**Tab 3: Saves**
- "Save Current Scene" button with name field
- List of saved scenes (from localStorage) showing name and date
  - Load button, Export button (download JSON), Delete button per entry
- "Import Scene" button (opens file picker for JSON)

### Bottom Tray (collapsible, always accessible)

- **Left section**: Recording controls
  - Record button (large, red circle icon; toggles to Stop icon during recording)
  - Text label showing recording state: "Ready" / "Recording" / "Paused"
- **Right section**: Camera controls
  - Camera mode toggle: three icon buttons (Fixed / Follow / Free Pan) — only one active at a time
  - Zoom slider (with - and + buttons at endpoints)

### Overlays and Modals

**Recording Countdown Overlay**: Full-canvas semi-transparent overlay showing "3... 2... 1..." as large centered text. Rendered by Phaser so it is visible to the user but triggered before MediaRecorder starts (the countdown does NOT appear in the recorded video).

**Preview Modal**: Centered modal with backdrop blur. Contains:
- Video player (recorded clip) with play/pause, scrub bar, volume, fullscreen toggle
- File info line: format (MP4/WebM), duration, estimated file size
- Primary action button: "Download" (prominent, full-width)
- Secondary action link: "Discard" (text link below the button)
- Tertiary action link: "Record Again" (starts a new take immediately)

**Onboarding Overlay**: Step-by-step guided overlay (see Journey 1). Spotlights UI elements. Non-blocking — user can interact with highlighted elements.

**Settings Modal** (accessible from gear icon):
- Audio: microphone device selection (if multiple), gain slider, enable/disable audio
- Video: recording format preference (Auto/MP4/WebM), bitrate quality (Low/Medium/High)
- Controls: deadzone slider, link to Action Binding Editor
- Display: canvas resolution, pixel scaling toggle
- About: version, attribution/licenses for open-source assets

**Gamepad Controls Reference Overlay**: Triggered from Help (?) button or a keyboard shortcut. Shows a visual gamepad diagram with all current button mappings labeled. Semi-transparent, dismissible, does not pause the canvas.

### Hidden During Recording

When recording is active (or when the user presses Select to hide UI):
- The sidebar, bottom tray, and top bar fade out
- The Phaser canvas fills the available space
- The recording indicator (red dot + timer) remains visible in the Vue layer (outside the Phaser canvas capture area, so it does NOT appear in the recorded video)
- A small "Show UI" floating button remains in a corner (low opacity) for emergency access

### Information Hierarchy

1. **The Canvas is always primary.** Every UI element serves the canvas. Panels exist to configure what happens on the canvas, then get out of the way.
2. **Recording controls are always accessible.** Whether via gamepad (Start button), bottom tray (Record button), or keyboard shortcut (R key), the user can start/stop recording from any context.
3. **Scene setup flows left-to-right conceptually:** choose characters (who), choose scene (where), configure camera (how), then record (go).
4. **The sidebar is a tool palette, not a navigation menu.** Users do not "navigate pages" — they switch between tool contexts (Characters, Scenes, Saves) while always seeing the live canvas.

---

## UX Recommendations

### R1: Make gamepad controls discoverable without requiring a keyboard or mouse

**Rationale**: The product's core identity is "pick up a gamepad and create." If the user must read documentation or mouse over tooltips to learn controls, the magic is broken.

**Recommendations**:
- On first gamepad connection, briefly flash a semi-transparent controller diagram overlay on the canvas (2-3 seconds) showing the basic controls: left stick = move, right stick = look, A = jump, X = attack, Start = record. This overlay fades automatically and does not block interaction.
- When the user presses a bumper (LB or RB) for the first time, show the emote wheel on-screen (Phaser-rendered, radial menu near the character) with the 4 emote options labeled on each face button position. The emote wheel remains visible as long as the bumper is held.
- The Help button (?) in the top bar should toggle a persistent gamepad reference overlay that can remain on-screen during play. This overlay should be rendered in the Vue layer so it does not appear in recordings when UI is hidden.
- Include a "Controls" quick reference as a tooltip on the gamepad status indicator (hover or click reveals the mapping).

### R2: Design the recording workflow for zero-surprise output

**Rationale**: A ruined recording wastes the user's creative energy. The system must make it nearly impossible to accidentally produce a bad recording.

**Recommendations**:
- **Auto-hide UI on record**: When the user initiates recording (via Start button or Record button), automatically hide all Vue UI elements if they are currently visible. Display a brief toast: "UI hidden for clean recording." This prevents the common mistake of recording with panels visible.
- **Pre-recording checklist (subtle)**: Before the countdown begins, display a 1-second status bar at the top of the canvas showing: mic status (active/muted/unavailable), recording format (MP4/WebM), and a green checkmark or red warning per item. This disappears before the countdown starts.
- **Countdown is pre-recording**: The 3-2-1 countdown must occur BEFORE the MediaRecorder starts capturing. The recorded video should start cleanly on the first frame of action, not on "1."
- **Recording indicator placement**: The red dot + timer should be rendered in the Vue overlay layer, which is captured by the user's eyes but NOT by the canvas captureStream. This ensures the recording indicator is visible during the session but absent from the output.
- **Stop confirmation**: When the user presses Start to stop recording, stop immediately (no confirmation dialog — the user's intent is clear). Proceed directly to the preview modal.
- **Preview before download**: Never auto-download. Always show the preview modal so the user can evaluate the take. The download button should display the file format and estimated size: "Download MP4 (12.4 MB)".

### R3: Handle "no gamepad connected" gracefully

**Rationale**: Many users will discover the tool before having a gamepad ready. If the tool appears broken without one, those users will not return.

**Recommendations**:
- **Landing state**: When no gamepad is detected, show a prominent but friendly banner across the top of the canvas: "Connect a gamepad and press any button to start puppeteering. [Use keyboard instead]". Below the banner, the canvas should still show the background and any placed characters (static, in idle animation) so the user can see what the tool does.
- **Keyboard fallback mapping**: WASD = movement, arrow keys = look-at direction, Space = jump, 1/2/3/4 = face button actions, Q/E = bumper emote modifiers, Enter = start/stop recording, Tab = hide/show UI. Display this mapping in the no-gamepad banner and in the Settings panel.
- **Gamepad connection event**: When a gamepad is connected mid-session (the `gamepadconnected` event fires), transition smoothly: dismiss the no-gamepad banner, flash the controller diagram overlay briefly, and enable gamepad input alongside keyboard. No page reload required.
- **Partial functionality without gamepad**: The user should be able to set up scenes (place characters, choose backgrounds, add props) using only the mouse, even without a gamepad. Only character real-time control requires gamepad or keyboard.

### R4: Design onboarding for first-time users that respects their time

**Rationale**: The target audience (content creators) has a low tolerance for lengthy tutorials. They want to create, not learn. But without understanding the gamepad-to-character-to-recording pipeline, they cannot create effectively.

**Recommendations**:
- **Interactive, not passive**: Each onboarding step should require the user to perform the action (move the stick, press a button, start a recording) before advancing. This is "learning by doing," not "reading instructions."
- **Skippable at every step**: A "Skip Tutorial" button is visible throughout. No step is mandatory.
- **Under 2 minutes total**: The entire onboarding should be completable in under 2 minutes for a user who follows along promptly.
- **Ends with a real artifact**: The onboarding culminates in the user downloading their first recording. This provides an immediate sense of accomplishment and demonstrates the full pipeline.
- **Re-accessible**: The tutorial can be replayed from Help -> "Show Tutorial." No need to clear localStorage.
- **No-gamepad variant**: If no gamepad is detected during onboarding, the tutorial adapts to show keyboard controls instead, with a note that gamepad provides the full experience.

### R5: Communicate available actions during gameplay

**Rationale**: Unlike a game where players learn controls over hours of play, Virtual Stage users need to know what is available NOW so they can direct performances in real-time.

**Recommendations**:
- **Emote wheel as a contextual menu**: When the user holds LB or RB, a radial menu appears near the character showing the 4 available emotes for that bumper, each labeled with the corresponding face button icon (A/B/X/Y). Releasing the bumper without pressing a face button dismisses the wheel with no action. This pattern is familiar from games (weapon wheels, emote wheels) and makes the full emote set discoverable.
- **Action hint on idle**: When the character is in the IDLE state for more than 5 seconds during a non-recording session, subtly show floating action hints near the character: small icons for the available actions (jump, attack, emote) with their button labels. These hints disappear when the character moves or an action is performed. During recording, these hints are suppressed.
- **State indicator (debug/practice mode)**: In a future iteration, offer an optional "director mode" that displays the character's current state (IDLE, WALK, RUN, etc.) as a small label above the character. Useful for practice but hidden during recording.

### R6: Protect recordings from tab visibility issues

**Rationale**: Background tab throttling in Chrome reduces requestAnimationFrame to approximately 1fps, which will produce an unusable recording if the user switches tabs. This is the most dangerous UX pitfall identified in the research.

**Recommendations**:
- **Immediate detection**: Use the Page Visibility API (`document.visibilitychange` event) to detect when the tab loses focus during recording.
- **Auto-pause with notification**: When the tab becomes hidden during recording, immediately pause the recording. When the tab regains visibility, display a prominent banner: "Recording was paused because you switched tabs. Resume recording?" with Resume and Stop buttons.
- **Pre-recording warning**: If the user's browser window is not in focus when they attempt to start recording (edge case), show a warning: "Make sure this tab stays in focus during recording."
- **Fullscreen suggestion**: On the first recording attempt, suggest: "For best results, press F11 for fullscreen mode. This prevents accidental tab switches." Show this as a non-blocking tooltip near the Record button, once per session.
- **No silent corruption**: Under no circumstances should the system continue recording while the tab is backgrounded and then present the user with a corrupted video. Pause or stop are the only acceptable responses.

### R7: Design the multi-gamepad assignment flow for clarity

**Rationale**: The most confusing moment in a multi-player session is figuring out which physical controller is "Gamepad 1" vs "Gamepad 2." The Gamepad API assigns indices based on connection order, which is not always intuitive.

**Recommendations**:
- **"Press to identify" interaction**: In the Character Panel's gamepad assignment dropdown, include an "Identify Controller" button. When clicked, the UI enters a listen state: "Press any button on the controller you want to assign." When a button press is detected, the system identifies the gamepad index and assigns it to that character. This is more intuitive than asking users to know their gamepad index.
- **Visual feedback on the canvas**: When a gamepad assignment changes, briefly flash the assigned character with a colored outline or glow matching a color assigned to that gamepad index (e.g., Gamepad 1 = blue, Gamepad 2 = red, Gamepad 3 = green, Gamepad 4 = yellow). This visual confirmation links the physical controller to the on-screen character.
- **Connection status always visible**: The top bar or sidebar should show a compact gamepad status: up to 4 small controller icons, each colored (active = green, disconnected = gray). Hovering/clicking reveals: controller name, assigned character, and a "Reassign" link.
- **Auto-assignment for single gamepad**: When only one gamepad is connected and one character is placed, auto-assign them. No user action needed.

### R8: Design for the "quick start" party scenario

**Rationale**: The Party Host persona has the lowest tolerance for setup friction. Four people standing around waiting for configuration will lose interest. The path from "everyone has a controller" to "we're recording" must be as short as possible.

**Recommendations**:
- **Quick Start option**: Offer a "Quick Start" button on the main screen (visible before any scene is set up). Quick Start: picks a random background, places 4 characters (one per detected gamepad, auto-assigned), and goes directly to the "ready to record" state. Total time: under 30 seconds.
- **Quick Controls card**: A single-screen visual reference (gamepad diagram with all mappings) that can be displayed as a full-canvas overlay. Accessible via Help -> "Show Controls" or a keyboard shortcut (F1). Designed to be glanced at by multiple people standing around a screen.
- **Minimal pre-recording friction**: For the party scenario, skip the VU meter check — just show a mic icon with a green/red indicator. Skip camera settings — default to Fixed, full stage view. Skip props — the fun is in character interaction, not scene composition.

### R9: Auto-save scene state to prevent work loss

**Rationale**: Browser-based tools are vulnerable to accidental tab closure, browser crashes, and power loss. Losing 20 minutes of scene composition to a crash is a retention-destroying experience.

**Recommendations**:
- **Auto-save every 30 seconds**: Silently serialize the current scene state to a dedicated localStorage key ("autosave"). No user action required.
- **Session restore on load**: When the application loads and finds an autosave, display a banner: "Restore your previous session?" with Restore and Start Fresh buttons. If the user does not respond within 10 seconds, the banner minimizes to a small "Restore" link in the sidebar.
- **Explicit save remains separate**: Auto-save does not replace the explicit Save Scene feature. Auto-save is a single rolling slot; explicit saves are named and persist until deleted.

### R10: Render recording indicator outside the capture area

**Rationale**: The recording indicator (red dot + timer) is essential for the user during recording, but it must NOT appear in the final video output.

**Recommendations**:
- The recording indicator should be rendered in the Vue overlay layer, which is positioned on top of the Phaser canvas visually but is not part of the canvas DOM element.
- Since `canvas.captureStream()` only captures the Phaser `<canvas>` element, any Vue-rendered overlay (including the recording indicator) will be visible to the user but absent from the recorded video.
- When the user hides the UI (Select button), the recording indicator should remain visible as an exception — it is the one UI element that persists during a hidden-UI recording.
- The indicator should be positioned in a corner that does not overlap with typical character action areas (top-right recommended, since most stage action occurs center and bottom).

---

## Accessibility Considerations

### Keyboard Fallback (REQ-F084)

- **Requirement**: All core character actions must be triggerable via keyboard for users who cannot use a gamepad. The keyboard mapping should cover: movement (WASD), look-at direction (arrow keys), all face button actions (mapped to number keys or other accessible keys), emote modifiers (Q/E for bumper equivalents), recording start/stop (Enter), and UI toggle (Tab).
- **Scope**: Keyboard provides functional parity for single-character control. Multi-character keyboard control is not expected (one keyboard cannot reasonably control 4 characters).
- **Implementation**: The GamepadService abstraction should accept both gamepad and keyboard input, normalizing them into the same action events. This keeps the character controller agnostic to input source.

### Color Contrast for UI Elements

- **Requirement**: All Vue UI text and interactive elements must meet WCAG 2.1 AA contrast ratio (4.5:1 for normal text, 3:1 for large text and UI components). This applies to panel text, button labels, dropdown options, and status indicators.
- **Recording indicator**: The red dot must be visible against any background. Use a red circle (#FF3333) with a white or dark outline ring to ensure contrast against both light and dark backgrounds.
- **Gamepad status colors**: Green (connected), gray (disconnected), yellow (warning) icons must be distinguishable by shape or label, not only by color. Add a tooltip or text label alongside each color indicator for colorblind users.
- **Canvas overlays**: The 3-2-1 countdown, action hints, and emote wheel labels displayed on the Phaser canvas must use high-contrast text (white text with a dark drop shadow or outline) to remain readable against any background image.

### Screen Reader Support for Non-Game UI

- **Requirement**: The Vue UI shell (sidebar panels, modals, settings, top bar) should be navigable by screen reader. Game canvas interaction (Phaser) is inherently visual and is exempt from screen reader requirements, but all configuration, setup, and management actions should be accessible.
- **Semantic HTML**: Use proper heading hierarchy (h2 for panel titles, h3 for sections), button elements for interactive controls, native select/input elements where possible, and ARIA labels for icon-only buttons.
- **Focus management**: When a modal opens (Preview Modal, Settings, Onboarding), trap focus within the modal. When it closes, return focus to the trigger element. Ensure tab order follows a logical flow through the sidebar and controls.
- **Live regions**: The recording status ("Recording... 02:34"), gamepad connection events ("Gamepad 2 connected"), and toast notifications should use `aria-live="polite"` regions so screen readers announce state changes without interrupting the user.
- **Alt text for asset thumbnails**: Background and character thumbnails in the sidebar should have descriptive alt text (e.g., "Castle hall background with stone walls and torches", "Knight character with sword and shield").

### Motor Accessibility

- **Large click targets**: All interactive UI elements (buttons, tabs, dropdowns) should be at minimum 44x44px to accommodate users with limited fine motor control.
- **No time-critical interactions**: The onboarding steps should not auto-advance. The countdown before recording gives a 3-second preparation window. No interaction in the application requires precise timing.
- **Sticky keys equivalent for emotes**: Consider allowing emote modifiers (LB/RB) to work as a toggle (press once to activate, press again to deactivate) in addition to the default hold behavior. This accommodates users who cannot hold a bumper while pressing a face button simultaneously. Expose this as a setting: "Emote modifier mode: Hold / Toggle."

### Reduced Motion

- **Requirement**: Respect the `prefers-reduced-motion` media query for Vue UI animations (panel slide-in/out, modal transitions, toast notifications). When reduced motion is preferred, replace animations with instant show/hide.
- **Canvas animations**: The Phaser canvas (character animations, background elements) is the core product and should not be suppressed by reduced motion preferences. The reduced motion setting applies only to the UI chrome, not the creative output.

### Cognitive Accessibility

- **Clear labeling**: Avoid jargon in the UI. Use "Record" not "Capture," "Download" not "Export," "Save Scene" not "Serialize." Every button label should describe exactly what happens when clicked.
- **Consistent layout**: The sidebar panel order (Characters, Scenes, Saves) should never change. Controls should always be in the same position. Predictability reduces cognitive load.
- **Undo/recovery**: Auto-save (R9) and the Preview modal's "Discard" option provide safety nets that reduce anxiety about making mistakes.

---

*End of Product & UX Analysis*
