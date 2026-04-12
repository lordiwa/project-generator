# Expert Analysis: Security & Compliance — Virtual Stage

**Project**: Virtual Stage
**Date**: 2026-04-12
**Agent**: Security & Compliance Expert
**Input Sources**: intake.md, expert-architect.md, expert-business-analyst.md

---

## Security Assessment

- **Data Classification**:
  - **Microphone Audio** (Sensitive): Live audio captured via `getUserMedia`. Contains user voice, ambient sounds, potentially identifiable speech. Processed entirely client-side through Web Audio API pipeline (GainNode -> AnalyserNode -> MediaStreamDestination). Never transmitted to a server unless embedded in a recording that the user explicitly uploads. Classified as **sensitive personal data** under GDPR (voice is biometric-adjacent).
  - **Recorded Video** (Sensitive): WebM/MP4 blobs containing canvas capture + microphone audio. Stored in browser memory as Blob chunks during recording, then offered for local download or optional Cloud Storage upload. May contain voice, and by extension, identifiable content. Up to 500MB per recording.
  - **Scene State Data** (Low sensitivity): JSON describing character positions, background selection, prop placement, camera settings. No personal data. Stored in localStorage and optionally Firestore.
  - **Firebase Auth Tokens** (Sensitive): Google OAuth tokens and Firebase ID tokens. Managed entirely by Firebase SDK. Stored in IndexedDB by Firebase. Contains user email, display name, profile photo URL.
  - **User Profile Data** (Low sensitivity): Display name, preferences (deadzone, codec). Stored in Firestore under authenticated user path.
  - **Asset Files** (Low sensitivity): Bundled pixel art sprites and backgrounds. Open-license. Future: user-uploaded custom sprites (PNG + JSON), which could contain arbitrary image content.

- **Threat Profile**:
  - **Attacker types**: (1) Opportunistic abusers uploading harmful content to Cloud Storage, (2) script kiddies probing Firebase misconfigurations, (3) supply chain attackers targeting npm dependencies (Phaser, mitt, Vue ecosystem), (4) malicious users crafting XSS payloads via scene names or imported scene JSON.
  - **What is at stake**: User privacy (microphone audio, voice recordings), Firebase resource abuse (storage quota exhaustion, unauthorized writes), content moderation liability (objectionable recordings stored on project infrastructure), and client-side integrity (XSS leading to session hijack or data exfiltration).
  - **What is NOT at stake**: No payment data, no health data, no financial records, no API keys exposed to users (Firebase client config is public by design). The application has no admin panel or privileged server-side logic beyond Firebase rules.

- **Compliance Scope**:
  - **GDPR**: Applies if EU users access the tool (likely, given global browser deployment). Voice audio is personal data. Google sign-in collects email/name.
  - **ePrivacy Directive**: Cookie/storage consent may be required for localStorage usage and Firebase Analytics.
  - **Browser Permissions Policy**: Microphone access governed by Permissions API and browser UX. No camera access required.
  - **Open-Source Licensing**: GPL-3.0, CC-BY-SA-3.0, and CC0 assets have attribution and copyleft requirements.
  - **Content Moderation**: Cloud-stored recordings may contain objectionable content. No legal obligation for pre-screening at this scale, but terms of service and abuse reporting are prudent.
  - **COPPA**: Not directly applicable (not targeted at children), but if children use the tool and sign in with Google, age verification is a consideration.
  - **SOC 2 / HIPAA / PCI**: Not applicable. No enterprise data, health data, or payment processing.

---

## Threat Model

### Microphone Access and Audio Capture

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| User unaware microphone is active during session | Information Disclosure | High | Medium | Clear visual indicator (VU meter, red mic icon) whenever AudioService has an active stream. Require explicit user action to initialize mic (not auto-init on page load). Provide mute/gain controls. |
| Microphone stream persists after user expects it stopped | Information Disclosure | High | Low | AudioService.destroy() calls `track.stop()` on all raw stream tracks. Verify browser mic indicator (tab icon) disappears. Implement `beforeunload` handler to stop all tracks. |
| Audio data inadvertently logged or transmitted | Information Disclosure | High | Low | Audio pipeline routes to MediaStreamDestination only (not `audioContext.destination`). No audio data is serialized, logged, or sent to any endpoint. Code review must verify no analytics or error-reporting library captures audio buffers. |
| Malicious browser extension intercepts getUserMedia stream | Tampering | Medium | Low | Out of scope (browser extension security is the user's responsibility). Document recommendation to disable unnecessary extensions during recording sessions. |

### Video Recording and Local Storage

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| Recording Blob exhausts browser memory causing data loss | Denial of Service | High | Medium | RecordingService enforces `memoryLimitMB` (default 450MB) and `maxDurationMs` (default 10 min). Memory warning emitted at threshold. Auto-stop on limit. |
| Blob URL leaked or accessible to other origins | Information Disclosure | Low | Very Low | `URL.createObjectURL()` returns opaque origins inaccessible cross-origin. Blob URLs are revoked after download via `URL.revokeObjectURL()`. |
| Tab visibility loss corrupts recording | Denial of Service | Medium | High | Page Visibility API monitoring with user notification. Option to auto-pause. Post-recording quality warning if visibility was lost. Already designed in RecordingService. |
| Recorded content contains copyrighted third-party audio (background music, etc.) | Repudiation | Low | Medium | Out of scope for technical controls. Terms of service should note user responsibility for recorded content. |

### Firebase Authentication Flow

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| Firebase API keys exposed in client bundle | Spoofing | Low | Certain (by design) | Firebase client configuration is intentionally public. Security is enforced by Firestore rules and Storage rules, not by secret keys. This is expected Firebase architecture. |
| OAuth token theft via XSS | Elevation of Privilege | High | Low | Firebase SDK stores tokens in IndexedDB with httpOnly-equivalent protections. Mitigate XSS risk via CSP headers and input sanitization (see Client-Side Security). |
| Account enumeration via Firebase Auth errors | Information Disclosure | Low | Low | Firebase Auth default error messages do not distinguish between "account not found" and "wrong password" for email/password. Google sign-in flow is handled by Google's consent screen. Not a significant risk for Google-only auth. |
| Unauthenticated access to cloud features | Spoofing | Medium | Low | Firestore rules require `request.auth != null` for all write operations and for reading non-public documents. Anonymous users can only use local features. |

### Firestore Scene Data Read/Write

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| User modifies another user's scene data | Tampering | Medium | Medium | Firestore rules enforce `request.auth.uid == resource.data.ownerId` for update/delete. Create requires authentication. Rule: only owners can modify their scenes. |
| Malicious scene JSON injected via Firestore | Tampering | Medium | Low | SceneStateService validates scene JSON structure on import (`validate()` method). Add strict schema validation using Zod or similar to reject unexpected fields, oversized strings, and prototype pollution payloads. |
| Scene data read by unauthorized users | Information Disclosure | Low | Low | Firestore rules gate read access: public scenes are readable by all, private scenes only by owner or `sharedWith` list. Scene data contains no sensitive personal information. |
| Excessive Firestore reads from enumeration | Denial of Service | Medium | Medium | Firestore does not expose collection listing by default. Queries require indexes. Rate limiting is handled by Firebase's built-in abuse protection. Add Firestore compound query limits. Consider App Check (see SEC-010). |
| sharedWith field manipulated by non-owner | Elevation of Privilege | Medium | Low | Current rules allow only owner to update. Verify that `sharedWith` modification is gated by the `update` rule requiring `request.auth.uid == resource.data.ownerId`. **Issue found**: The architect's rules allow any authenticated user to `create` a scene and set arbitrary `ownerId`. Fix: validate `request.resource.data.ownerId == request.auth.uid` on create. |

### Cloud Storage Video Upload/Download

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| User uploads non-video file disguised as video | Tampering | Medium | Medium | Storage rules validate `request.resource.contentType.matches('video/.*')`. However, content-type headers can be spoofed client-side. Add a Cloud Function trigger on upload to validate actual file content (magic bytes) if abuse becomes a concern. For MVP, content-type validation is proportionate. |
| Storage quota exhaustion by a single user | Denial of Service | Medium | Medium | Storage rules enforce 500MB max per file. Add per-user quota enforcement: either via a Cloud Function that checks total usage before allowing upload, or by tracking upload count in Firestore and checking in rules. |
| Uploaded recordings contain illegal/harmful content | Tampering | Medium | Low (at current scale) | Terms of service + abuse reporting mechanism. At scale, integrate content moderation (Google Cloud Video Intelligence API or manual review queue). For MVP, this is a policy control, not a technical one. |
| Public read rule allows unrestricted download of all recordings | Information Disclosure | High | Medium | **Issue found**: The architect's Storage rules have `allow read: if true` for recordings, meaning anyone who guesses or enumerates a storage path can download any recording. Fix: change to `allow read: if request.auth.uid == userId` for private recordings, and use Firebase-generated signed URLs with expiration for public sharing. Alternatively, gate public reads on a Firestore `isPublic` flag checked via a Cloud Function. |
| Large file upload used as denial-of-service vector | Denial of Service | Medium | Low | 500MB limit already in rules. Firebase Storage has built-in rate limiting. Consider reducing default max to 200MB and allowing higher limits for authenticated power users. |

### Asset Loading (Bundled + Future User Uploads)

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| Malicious SVG/PNG uploaded as user asset containing embedded script | Tampering | High | Medium | Storage rules restrict to `image/png` and `application/json`. SVG is not allowed — this is correct and must remain so. PNG cannot execute scripts. JSON animation files must be parsed with `JSON.parse()` (safe) and validated against a strict schema. |
| Oversized asset upload causing storage abuse | Denial of Service | Low | Low | 10MB limit per asset file in Storage rules. Proportionate for sprite sheets. |
| Asset manifest poisoning via crafted JSON | Tampering | Medium | Low | Bundled manifest is part of the deployed build and not user-editable. User-uploaded assets in the future should not modify the global manifest — they should be loaded into a separate user asset namespace. Validate all asset JSON before passing to Phaser loader. |
| Phaser loader processes malicious image triggering browser vulnerability | Tampering | Low | Very Low | Phaser uses standard browser image decoding (`<img>` element / `createImageBitmap`). Browser image decoders are hardened. Keep browser recommendations current. |

### Client-Side Application (XSS, Supply Chain)

| Threat | Category | Impact | Likelihood | Mitigation |
|--------|----------|--------|------------|------------|
| XSS via scene name or prop data rendered in Vue templates | Tampering | High | Medium | Vue 3 auto-escapes template interpolation (`{{ }}`). Verify no use of `v-html` with user-supplied data. Scene names and metadata must be text-only — enforce max length and character whitelist on input. |
| XSS via imported scene JSON containing script payloads | Tampering | Medium | Low | SceneStateService.validate() checks structure but not content. Add string sanitization for all string fields in imported scene JSON (name, characterId, propId, etc.). Reject strings containing `<script>`, `javascript:`, event handlers, or strings exceeding reasonable length (e.g., 200 chars for names). |
| Dependency supply chain attack (compromised npm package) | Tampering | High | Low | Use `npm audit` in CI/CD. Pin exact dependency versions in `package-lock.json`. Review Phaser, mitt, Vue, Pinia, and Vite releases for security advisories. Consider using Socket.dev or Snyk for continuous monitoring. Phaser 3.90.0 being EOL means any discovered vulnerability has no upstream fix path — monitor Phaser security discussions. |
| Prototype pollution via JSON.parse of untrusted scene data | Elevation of Privilege | Medium | Low | `JSON.parse()` itself is safe from prototype pollution. However, if parsed objects are spread into other objects with `Object.assign` or spread operators, `__proto__` keys could pollute prototypes. Validate and strip `__proto__`, `constructor`, and `prototype` keys from imported JSON. |
| Sensitive data leaked via browser developer tools or console | Information Disclosure | Low | Low | Do not log audio data, auth tokens, or recording blobs to console. In production builds, strip `console.log` statements or configure log levels. Firebase SDK handles token storage securely. |

---

## Security Requirements

- **SEC-001**: The application shall not initialize microphone capture (`getUserMedia`) without explicit user action (button click or gamepad button press). Auto-initialization on page load is prohibited. (Priority: **Must**)

- **SEC-002**: The application shall display a persistent, unambiguous visual indicator whenever the microphone stream is active, including a VU meter or mic icon in the UI and cooperation with the browser's native tab-level mic indicator. (Priority: **Must**)

- **SEC-003**: The application shall stop all active MediaStream tracks (`track.stop()`) when the user navigates away, closes the tab, or explicitly disables the microphone. A `beforeunload` event handler shall ensure cleanup. (Priority: **Must**)

- **SEC-004**: All Firestore security rules shall validate that `request.resource.data.ownerId == request.auth.uid` on document creation to prevent users from creating documents attributed to other users. (Priority: **Must**)

- **SEC-005**: Cloud Storage read rules for recordings shall not use `allow read: if true`. Private recordings shall require `request.auth.uid == userId`. Public sharing shall use time-limited signed URLs generated by a Cloud Function. (Priority: **Must**)

- **SEC-006**: The application shall deploy Content Security Policy headers via `firebase.json` that restrict `script-src` to `'self'` (plus Firebase SDK domains and any CDN origins), disallow `unsafe-inline` for scripts (nonce-based or hash-based if needed), and restrict `connect-src` to Firebase endpoints, the application origin, and asset CDN origins. (Priority: **Must**)

- **SEC-007**: All imported scene JSON files shall be validated against a strict schema (Zod or equivalent) that rejects unknown fields, enforces string length limits, strips prototype-pollution keys (`__proto__`, `constructor`, `prototype`), and validates all expected types before the data is used by the application. (Priority: **Must**)

- **SEC-008**: The application shall never render user-supplied strings using `v-html` in Vue templates. All user-visible text (scene names, character names, metadata) shall use Vue's default text interpolation with auto-escaping. (Priority: **Must**)

- **SEC-009**: The CI/CD pipeline shall run `npm audit` on every build and fail the build on high or critical severity vulnerabilities. Dependency versions shall be pinned via `package-lock.json`. (Priority: **Must**)

- **SEC-010**: Firebase App Check shall be enabled to restrict Firebase API access to requests originating from the legitimate application, mitigating automated abuse of Firestore and Cloud Storage endpoints. (Priority: **Should**)

- **SEC-011**: Per-user Cloud Storage quota shall be enforced, either via Firestore-tracked counters checked in Storage rules or via a Cloud Function upload gate, to prevent a single user from exhausting storage quota. Initial limit: 2GB per user. (Priority: **Should**)

- **SEC-012**: The application shall include a visible attribution panel listing all open-license assets used, with license type, author, and source URL, satisfying GPL-3.0 and CC-BY-SA-3.0 requirements. This panel shall be accessible from the main UI at all times. (Priority: **Must**)

- **SEC-013**: SVG files shall never be accepted as user-uploaded assets. Only `image/png` and `application/json` content types shall be permitted in Storage rules for the user-uploads path. (Priority: **Must**)

- **SEC-014**: Recorded video Blob URLs shall be revoked via `URL.revokeObjectURL()` after download or discard to prevent memory leaks and stale URL references. (Priority: **Should**)

- **SEC-015**: Firebase Hosting shall be configured with security headers: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` (or `SAMEORIGIN` if embedding is needed), `Referrer-Policy: strict-origin-when-cross-origin`, and `Permissions-Policy: microphone=(self)`. (Priority: **Must**)

- **SEC-016**: The application shall present a privacy notice before first microphone activation, informing the user that audio will be captured, processed locally, and (if recording) embedded in the video file. This notice shall be dismissible but not bypassable. (Priority: **Should**)

- **SEC-017**: All string fields in Firestore documents (scene name, user display name) shall enforce maximum length limits in Firestore security rules using `request.resource.data.name.size() <= 200`. (Priority: **Should**)

- **SEC-018**: Production builds shall not include source maps deployed to Firebase Hosting. Source maps shall be generated for error tracking services only (uploaded to Sentry/Crashlytics separately). (Priority: **Should**)

---

## Recommended Controls

### Authentication

- **Use Firebase Auth with Google sign-in only for MVP**. Google's OAuth 2.0 flow handles credential security, token refresh, and session management. No custom password storage required. This eliminates an entire class of credential-related vulnerabilities.
- **Enforce authentication for all write operations** in Firestore and Cloud Storage rules. Anonymous/unauthenticated users can use the local-only tool but cannot interact with cloud resources.
- **Enable Firebase App Check** (reCAPTCHA Enterprise provider for web) to verify that requests to Firebase services originate from the legitimate app instance. This prevents automated scripts from directly calling Firestore/Storage APIs with stolen config.
- **Do not implement "remember me" or custom session management**. Firebase SDK handles token persistence in IndexedDB with appropriate expiration and refresh logic. Do not override this.

### Authorization

- **Firestore Rules** (corrected from architect's draft):

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // User profile: owner-only read/write
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }

    // Scenes: owner creates, owner edits, public or authorized reads
    match /scenes/{sceneId} {
      allow read: if resource.data.isPublic == true
                  || (request.auth != null && request.auth.uid == resource.data.ownerId)
                  || (request.auth != null && request.auth.uid in resource.data.sharedWith);
      allow create: if request.auth != null
                    && request.resource.data.ownerId == request.auth.uid
                    && request.resource.data.name is string
                    && request.resource.data.name.size() <= 200;
      allow update: if request.auth != null
                    && request.auth.uid == resource.data.ownerId
                    && request.resource.data.ownerId == resource.data.ownerId; // prevent owner transfer
      allow delete: if request.auth != null
                    && request.auth.uid == resource.data.ownerId;
    }

    // Recordings: owner creates, owner manages, public or owner reads
    match /recordings/{recordingId} {
      allow read: if resource.data.isPublic == true
                  || (request.auth != null && request.auth.uid == resource.data.ownerId);
      allow create: if request.auth != null
                    && request.resource.data.ownerId == request.auth.uid;
      allow update: if request.auth != null
                    && request.auth.uid == resource.data.ownerId
                    && request.resource.data.ownerId == resource.data.ownerId;
      allow delete: if request.auth != null
                    && request.auth.uid == resource.data.ownerId;
    }
  }
}
```

- **Cloud Storage Rules** (corrected):

```javascript
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {

    // Recordings: owner writes, reads require auth (public sharing via signed URLs)
    match /recordings/{userId}/{allPaths=**} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null
                   && request.auth.uid == userId
                   && request.resource.size < 500 * 1024 * 1024
                   && request.resource.contentType.matches('video/.*');
    }

    // User-uploaded assets: owner writes, authenticated reads
    match /assets/user-uploads/{userId}/{allPaths=**} {
      allow read: if request.auth != null;
      allow write: if request.auth != null
                   && request.auth.uid == userId
                   && request.resource.size < 10 * 1024 * 1024
                   && (request.resource.contentType == 'image/png'
                       || request.resource.contentType == 'application/json');
    }
  }
}
```

- **Public sharing pattern**: When a user marks a recording as public, generate a time-limited signed URL via a Cloud Function. Do not expose raw Storage paths. This prevents enumeration and gives revocation control.

### Data Protection

- **In-Transit**: All Firebase services enforce HTTPS by default. Firebase Hosting serves over HTTPS with automatic SSL. No additional TLS configuration needed.
- **At-Rest**: Firestore and Cloud Storage encrypt data at rest by default (Google-managed keys). No additional encryption configuration needed for this threat profile.
- **Microphone Data Handling**:
  - Audio stream is routed through Web Audio API nodes client-side only. The processed stream goes to MediaStreamDestination for recording — it is never sent to a server endpoint, analytics service, or logging system.
  - When the user stops recording or disables the mic, all tracks are stopped and the AudioContext is closed. Verify no dangling references retain the stream.
  - The `beforeunload` handler must call `AudioService.destroy()` and `RecordingService.destroy()` to guarantee cleanup.
- **Blob Lifecycle**: Recording Blob chunks exist only in browser memory. On download, a temporary Blob URL is created and immediately revoked after the download link is clicked. On discard, the chunks array is emptied and garbage collected. On Cloud Storage upload, the Blob is transmitted over HTTPS and then discarded locally.
- **localStorage**: Scene state in localStorage is unencrypted but contains no sensitive data (character positions, background IDs). This is acceptable for the data classification level.

### Client-Side Security

- **Content Security Policy** (configured in `firebase.json` headers):

```json
{
  "hosting": {
    "headers": [
      {
        "source": "**",
        "headers": [
          {
            "key": "Content-Security-Policy",
            "value": "default-src 'self'; script-src 'self' https://apis.google.com https://www.gstatic.com; style-src 'self' 'unsafe-inline'; img-src 'self' blob: data: https://firebasestorage.googleapis.com; media-src 'self' blob:; connect-src 'self' https://*.googleapis.com https://*.firebaseio.com wss://*.firebaseio.com https://firebasestorage.googleapis.com; frame-src https://accounts.google.com https://*.firebaseapp.com; font-src 'self'; object-src 'none'; base-uri 'self';"
          },
          {
            "key": "X-Content-Type-Options",
            "value": "nosniff"
          },
          {
            "key": "X-Frame-Options",
            "value": "DENY"
          },
          {
            "key": "Referrer-Policy",
            "value": "strict-origin-when-cross-origin"
          },
          {
            "key": "Permissions-Policy",
            "value": "microphone=(self), camera=(), geolocation=(), payment=()"
          }
        ]
      }
    ]
  }
}
```

- **Dependency Auditing**: Run `npm audit --audit-level=high` in GitHub Actions CI. Integrate Socket.dev or Snyk for PR-level dependency risk scoring. Pin all dependencies with exact versions. Review lockfile diffs on dependency updates.
- **Input Sanitization**: Validate and sanitize all user-provided strings (scene names, character names) on input — enforce character whitelists (`[a-zA-Z0-9 _-]`), length limits (200 chars max), and reject control characters. For imported scene JSON, use Zod schema validation with `.strip()` to remove unknown keys.
- **No `v-html`**: Enforce via ESLint rule (`vue/no-v-html`). All user data rendered via `{{ }}` interpolation only.
- **Subresource Integrity (SRI)**: If loading any scripts from CDNs (unlikely with Vite bundling, but if Firebase SDK is loaded via `<script>` tag), use SRI hashes.

### Infrastructure

- **Firebase Project Configuration**:
  - Enable App Check for Firestore and Cloud Storage.
  - Restrict authorized domains in Firebase Auth to the production domain only (remove `localhost` before production launch; re-add only for development builds).
  - Enable Firebase Hosting preview channels for PR previews, but restrict Auth domains to prevent preview channels from being used for phishing.
  - Set Firestore and Cloud Storage billing alerts to detect unexpected usage spikes.

- **CI/CD Security**:
  - Firebase deploy token stored as GitHub Actions encrypted secret.
  - No Firebase admin credentials in the client bundle.
  - Production deployments gated behind a manual approval step or protected branch.
  - Source maps uploaded to error tracking service only, not deployed to Hosting.

- **Monitoring**:
  - Firebase Security Rules playground testing before every rules deployment.
  - Firebase Alerts for authentication anomalies (spike in failed sign-ins, new sign-in from unusual geography).
  - Cloud Storage usage monitoring to detect abuse patterns.

---

## Compliance Checklist

### Browser Permission Policies

- [x] Microphone access requested only on explicit user action (not on page load) — SEC-001
- [ ] Privacy notice displayed before first microphone activation explaining what is captured and how it is used — SEC-016
- [ ] `Permissions-Policy` header restricts `microphone` to `(self)` and denies `camera`, `geolocation`, `payment` — SEC-015
- [ ] Browser's native microphone indicator (tab icon) is verified to activate/deactivate correctly during AudioService lifecycle
- [ ] No camera access requested (application does not use camera)

### GDPR Considerations

- [ ] **Privacy Policy**: Publish a privacy policy accessible from the application footer covering: (a) what data is collected (Google profile info, scene data, recordings), (b) how it is processed (client-side audio/video, Firebase storage), (c) data retention periods, (d) user rights (access, deletion, portability), (e) legal basis (consent for mic access, legitimate interest for service provision)
- [ ] **Data Minimization**: Collect only what is needed — Google sign-in profile (name, email, photo URL) for display purposes. Do not request additional OAuth scopes. Do not store raw microphone audio independently from recordings.
- [ ] **Right to Deletion**: Implement account deletion that removes the user's Firestore documents (profile, scenes, recordings metadata) and Cloud Storage files (recordings, uploaded assets). Firebase Auth account deletion via `user.delete()`.
- [ ] **Cookie/Storage Consent**: If Firebase Analytics or Performance Monitoring is enabled, display a cookie consent banner for EU users. localStorage for scene state is arguably "strictly necessary" for functionality but Analytics is not.
- [ ] **Data Processing Agreement**: Google Cloud / Firebase is the data processor. Google's standard DPA covers GDPR requirements for Firebase services.
- [ ] **Cross-Border Transfer**: Firebase stores data in the configured project region. Select a region appropriate for the user base (e.g., `europe-west1` if targeting EU users, or `us-central1` with Standard Contractual Clauses).

### Asset Licensing Compliance

- [ ] **Attribution Panel**: Implement a visible "Credits" or "Attributions" section accessible from the main UI listing every third-party asset with: asset name, author/creator, license type (CC0, CC-BY-SA-3.0, GPL-3.0), and source URL — SEC-012
- [ ] **CC-BY-SA-3.0 Requirements**: Attribution must include (a) creator name, (b) license name and link, (c) indication of modifications if the asset was altered. The attribution must be "reasonable" for the medium — an in-app credits panel satisfies this.
- [ ] **GPL-3.0 Art Assets**: GPL on art assets (LPC originals) requires attribution and that derivative works using those assets are distributed under GPL-compatible terms. Clarify: if users create recordings using GPL assets, the recording output is not a "derivative work" of the sprite sheet (it is a rendered output, similar to a screenshot). However, if users redistribute the sprite sheets themselves, GPL applies. Document this interpretation clearly.
- [ ] **License Metadata in AssetService**: The `LicenseInfo` type in `AssetManifest` correctly captures license type, attribution, and URL per asset. Ensure this data is populated for all bundled assets and displayed in the attribution panel.
- [ ] **User-Uploaded Assets**: When user-uploaded custom assets are supported, the upload flow should include a license declaration (e.g., "I have the right to use this asset"). Terms of service should indemnify the platform.

### Content Moderation for Cloud-Stored Recordings

- [ ] **Terms of Service**: Draft ToS that prohibit uploading illegal, abusive, or infringing content. Users must agree before first Cloud Storage upload.
- [ ] **Abuse Reporting**: Implement a "Report" mechanism for publicly shared recordings. At MVP scale, this can be an email link. At scale, integrate a moderation queue.
- [ ] **Content Review (Future)**: If public sharing grows, evaluate Google Cloud Video Intelligence API for automated content moderation (violence, explicit content detection). Not required at MVP.
- [ ] **Takedown Process**: Document a process for responding to DMCA/content takedown requests. At minimum: ability to delete specific recordings from Cloud Storage and Firestore by admin.

---

## Security Testing Requirements

### Static Analysis

- **ESLint Security Rules**: Enable `eslint-plugin-security` and `vue/no-v-html` rule. Run in CI on every PR. Catches common patterns: `eval()`, `innerHTML`, `document.write()`, regex DoS.
- **Dependency Audit**: `npm audit` in CI. Fail on high/critical. Weekly scheduled audit run with notifications.
- **Source Map Verification**: CI check that production build output does not include `.map` files in the Hosting deployment directory.

### Firebase Rules Testing

- **Unit Tests for Security Rules**: Use the Firebase Emulator Suite with `@firebase/rules-unit-testing` to write test cases for:
  - Unauthenticated user cannot read/write any protected path
  - Authenticated user can only create documents with their own `ownerId`
  - Authenticated user cannot modify another user's scene or recording
  - Owner cannot transfer ownership (ownerId immutable on update)
  - Public scenes/recordings are readable by any authenticated user
  - Storage upload rejects files exceeding size limits
  - Storage upload rejects non-video content types for recordings
  - Storage upload rejects non-PNG/JSON content types for assets
  - String length limits are enforced on scene names
- **Rules Deployment Pipeline**: Run rules tests before every `firebase deploy --only firestore:rules,storage` deployment. Block deploy on test failure.

### Client-Side Security Testing

- **CSP Validation**: After deployment, verify CSP headers are applied correctly using browser developer tools and `securityheaders.com`. Confirm no CSP violation reports in the console for normal application usage.
- **XSS Testing**: Manually test scene import with crafted JSON containing `<script>` tags, `javascript:` URLs, event handler attributes, and oversized strings. Verify all are rejected or safely escaped.
- **Prototype Pollution Testing**: Import scene JSON containing `__proto__`, `constructor.prototype`, and `toString` keys. Verify they are stripped or ignored.

### Microphone and Recording Privacy Testing

- **Mic Lifecycle Test**: Verify that after AudioService.destroy(), the browser's tab-level microphone indicator disappears. Test on Chrome, Edge, and Firefox.
- **Tab Close Cleanup**: Verify that closing the tab while recording stops all MediaStream tracks (no lingering mic access). Test via `beforeunload` handler.
- **Blob Cleanup Test**: After discarding a recording, verify via Chrome DevTools Memory panel that Blob data is garbage collected and memory returns to baseline.

### Penetration Testing (Pre-Launch of Cloud Features)

- **Firebase Rules Probe**: Use the Firebase REST API directly (bypassing the app) to attempt unauthorized Firestore reads/writes and Storage uploads. Verify rules block all unauthorized access.
- **Storage Enumeration**: Attempt to list or guess paths in Cloud Storage to access other users' recordings. Verify that unauthenticated or unauthorized requests are rejected.
- **Rate Limiting Validation**: Send rapid bursts of Firestore writes and Storage uploads from a single authenticated account. Verify Firebase's built-in abuse protection activates and that per-user quotas are enforced.

---

## Issues Found in Architecture Review

### ISSUE-1: Firestore Create Rule Missing ownerId Validation (Severity: High)

The architect's Firestore rules allow `allow create: if request.auth != null` for scenes and recordings without validating that the `ownerId` field matches the authenticated user. A malicious user could create documents attributed to another user's ID.

**Fix**: Add `&& request.resource.data.ownerId == request.auth.uid` to all create rules. See corrected rules in the Authorization section above.

### ISSUE-2: Cloud Storage Public Read on Recordings (Severity: High)

The architect's Storage rules use `allow read: if true` for the recordings path. This means any user (or unauthenticated script) who knows or guesses a storage path can download any recording. Given that recordings contain voice audio, this is a privacy concern.

**Fix**: Change to `allow read: if request.auth != null && request.auth.uid == userId`. For public sharing, generate time-limited signed URLs via a Cloud Function that checks the Firestore `isPublic` flag.

### ISSUE-3: Firestore Update Rule Allows ownerId Change (Severity: Medium)

The architect's Firestore rules allow the owner to update any field, including `ownerId`. A compromised or malicious client could transfer document ownership, potentially locking out the original creator.

**Fix**: Add `&& request.resource.data.ownerId == resource.data.ownerId` to update rules to make `ownerId` immutable after creation.

### ISSUE-4: No String Length Validation in Firestore Rules (Severity: Low)

Scene names and other string fields in Firestore documents have no length limits in the security rules. A malicious client could write extremely large strings, consuming storage and potentially causing rendering issues.

**Fix**: Add `request.resource.data.name.size() <= 200` (or appropriate limit) to create and update rules.

---

*End of Security & Compliance Analysis*
