# OmniPlayer v1.0.0 – Architecture & Engineering Log

## App Overview & Core Problem

**Name:** OmniPlayer
**Primary function:** Native Apple Silicon macOS desktop media player for personal and controlled internal use. It plays AVFoundation-compatible media directly and transparently converts selected unsupported codecs into native-compatible ProRes media using bundled FFmpeg/FFprobe binaries.

### Production Problem Solved
* Removed codec incompatibility friction in native macOS playback, particularly for QuickTime Animation (`qtrle`), FFV1, MPEG-2, and other non-native codecs.
* Eliminated manual media inspection, command-line transcoding, output management, and reopening of converted files.
* Operated within a restricted corporate environment with no Homebrew, Xcode, VLC, mpv, runtime downloads, or end-user dependency installation.
* Consolidated native and fallback media into one ordered playlist instead of requiring separate playback and conversion tools.
* Replaced fragile manual packaging with reproducible build, validation, release ZIP, checksum, architecture, and signature workflows.

---

## Tech Stack & Systems Architecture

* **Language:** Swift 6.3.3
* **Platform:** macOS 26+, Apple Silicon `arm64`
* **UI layer:** AppKit
* **Playback layer:** AVKit, AVFoundation, `AVPlayer`, `AVPlayerView`
* **Audio processing:** MediaToolbox `MTAudioProcessingTap`
* **Animation/rendering:** Core Animation vector layers using `CAShapeLayer`
* **Media tooling:** FFmpeg 7.1.1 and FFprobe, statically compiled locally from pinned official source
* **Build system:** Terminal-based `swiftc` compilation with Apple Command Line Tools
* **Packaging:** Manually assembled `.app` bundle with generated `Info.plist`
* **Signing:** Ad-hoc `codesign`; Developer ID signing and notarization are not currently implemented
* **Testing:** Standalone Swift smoke-test executables coordinated by shell scripts
* **Release format:** Versioned ZIP plus SHA-256 checksum

### Core Modules
* **`FFprobeService`** — asynchronous primary-video-codec detection
* **`MediaRoutingPolicy`** — centralized codec-to-playback-route policy
* **`FFmpegConversionProfile`** — deterministic FFmpeg argument generation
* **`FFmpegConversionService`** — asynchronous conversion, cancellation, cleanup, and failure handling
* **`PlaylistStore`** — ordered playlist state, navigation, removal, clearing, and reordering
* **`PlaylistSidebarView`** — compact AppKit playlist interface
* **`RecentMediaStore`** — persistent, deduplicated ten-item recent-file history
* **`AudioLevelMonitor`** — live post-effects PCM capture and level extraction
* **`ThreeBandAudioAnalyzer`** — bass, mid, and high frequency separation
* **`MediaDropPlayerView`** — playback surface, drag-and-drop, and vector visualization
* **`main.swift`** — application orchestration, menus, playback lifecycle, and shared state integration

### External Dependencies
* **FFmpeg 7.1.1**
* **FFprobe 7.1.1**
* *No third-party Swift packages*
* *No runtime network services, local AI models, Python runtime, or package manager dependencies*

---

## Engineering Highlights & Deep-Dive (The "How It Works")

### Media Processing Pipeline
1. Media enters through File → Open, Open Recent, multi-file selection, drag-and-drop, or playlist controls.
2. `PlaylistStore` maintains ordered file URLs and the current selection.
3. Each playback request receives a unique request identifier.
4. `FFprobeService` inspects the first video stream asynchronously.
5. `MediaRoutingPolicy` selects one of three routes:
   * **Native playback:** AVFoundation-supported or unknown codecs are attempted directly.
   * **Alpha-preserving conversion:** `qtrle` is converted to ProRes 4444 using `yuva444p10le`.
   * **Standard conversion:** VP8, VP9, AV1, Theora, MPEG-2, MPEG-4, WMV3, VC-1, and FFV1 are converted to ProRes 422 HQ using `yuv422p10le`.
6. FFmpeg runs on a dedicated user-initiated dispatch queue.
7. Conversion output is written to a UUID-named temporary `.mov`.
8. Successful output is passed to `AVPlayer`; failed or cancelled output is deleted.
9. Previous temporary media is removed when replaced, cleared, or during shutdown.
10. Playback completion advances to the next item in the current reordered playlist.

### Concurrency and Lifecycle Safety
* FFprobe and FFmpeg operations run off the main thread.
* Completion handlers are marshalled back to the main thread before UI or playback updates.
* Request IDs prevent stale asynchronous inspection or conversion results from replacing a newer user selection.
* Conversion state is protected by `NSLock`.
* Cancellation first sends termination, then uses a one-second `SIGKILL` fallback for unresponsive FFmpeg processes.
* Cancellation, playlist clearing, media replacement, and shutdown invalidate active work safely.
* Expected cancellation is logged at `INFO`; operational failures are logged at `ERROR`.

### Native Codec Fallback Architecture
* Routing policy is isolated from playback orchestration and covered by smoke tests.
* Conversion arguments are profile-driven rather than embedded in UI logic.
* Optional source audio is mapped and converted to PCM for macOS-compatible output.
* FFmpeg and FFprobe are resolved relative to `Bundle.main.resourceURL`, preventing hardcoded machine paths and eliminating external installation requirements.
* Bundled application and media tools are validated as `arm64` Mach-O executables.

### Playlist and UX Architecture
* The playlist is an ordered in-memory URL model with safe boundary handling.
* Current-item identity is preserved when rows are reordered.
* The AppKit sidebar starts hidden and toggles through View or `⌘P`.
* Supported actions include append-without-autoplay, remove current, clear, double-click playback, Previous/Next, automatic advancement, and drag-to-reorder.
* Open Recent selects an existing playlist entry or appends it without duplication.
* Focus-independent Spacebar handling uses an AppKit local key-event monitor and suppresses key-repeat instability.

### Real-Time Audio Visualization
* `MTAudioProcessingTap` captures post-effects PCM from the active `AVPlayerItem` audio track without replacing AVPlayer.
* Audio processing occurs inside the media tap; the UI reads synchronized levels on a 30 Hz main-run-loop timer.
* PCM samples are downmixed safely for analysis.
* `ThreeBandAudioAnalyzer` uses lightweight stateful crossover filters:
  * **Bass:** ~20–250 Hz
  * **Mids:** ~250 Hz–4 kHz
  * **Highs:** ~4–16 kHz
* Synthetic tones (100 Hz, 1 kHz, 8 kHz) validate band dominance; silence validates zero response.
* **Visual mapping:** Bass drives the central orange ring, Mids drive inner arcs, Highs drive outer arcs.
* Core Animation transactions smooth scale and opacity changes.
* Dash phases animate on exact pattern boundaries to avoid visible loop jumps.
* Layer bounds and position are managed independently of transforms, preventing visualization drift during resizing.

### Packaging and Release Hardening
* `VERSION` is the single source of truth for application and build numbers.
* Packaging generates the application bundle, embeds FFmpeg tools, validates `Info.plist`, and applies an ad-hoc signature.
* **Release creation:**
  1. Runs the complete automated validation suite.
  2. Builds and packages the app.
  3. Creates a client-clean versioned ZIP.
  4. Generates a SHA-256 checksum.
  5. Extracts the ZIP into a clean temporary directory.
  6. Revalidates structure, metadata, version, bundle identifier, minimum macOS version, executable architecture, checksum, and signature.

---

## Quantifiable Impact / Workflow Transformation
* **Reduced cancellation lag:** Brought unresponsive FFmpeg cancellation recovery down to ~1–2 seconds.
* **DevOps Automation:** Consolidated compilation, packaging, architecture checks, signing, and ZIP creation into repeatable one-command workflows.
* **Zero Dependency Setup:** Eliminated all end-user runtime installation steps (no Python, Homebrew, or network downloads required).
* **Workflow Consolidation:** Replaced the manual workflow of codec inspection, command-line conversion, cleanup, and reopening media.
* **Native Performance:** Audio visualization updates at 30 Hz while computationally lightweight filtering remains inside the native playback pipeline.
* **Production Validation:** Final v1.0.0 validation covered 32 tracked project files and real playback passed for native H.264, alpha-preserving `qtrle`, MPEG-2, and FFV1 workflows.
