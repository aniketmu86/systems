# Subtitle Studio v6 Roadmap

Current branch: `v5-finalization`  
Current baseline commit before roadmap creation: `9e26dd52`  
Current product state: v5.0.1 replacement hotfix package / v6 polish preparation

Latest important pre-v6 commits:

```text
49df56f3 Fix burned subtitle quote and background rendering
981556d9 Isolate launcher in ship package
```

---

## v6 Goal

Subtitle Studio v6 should make the existing working app feel more reliable, precise, and professional.

The v6 direction is:

```text
Precision first.
Preview trust second.
Workflow polish third.
Packaging last.
```

This is not a rewrite.  
This is not a broad packaging refactor.  
This is not a rebuild of already-working features.

---

## v5.0.1 Hotfix Work Completed Before v6

Before v6 development resumed, two release-critical fixes were completed and pulled into a v5.0.1 replacement package.

### Completed commits

```text
49df56f3 Fix burned subtitle quote and background rendering
981556d9 Isolate launcher in ship package
```

---

### Completed export rendering hotfix

Burned subtitle export now supports:

- Correct opening and closing curly double quotes
- Correct opening and closing curly single quotes
- Curly apostrophes for contractions and possessives
- Export-only smart typography, without mutating editor subtitle text
- Multi-line subtitle background boxes rendered as one merged square-corner shape
- No visible dark overlap from stacked transparent subtitle boxes
- T-shaped and inverted-T-shaped merged boxes depending on line width
- Left, center, and right subtitle alignment preserved
- Text vertically centered inside the subtitle background shape
- Uniform absolute padding tuned visually

This work was completed in:

```text
tabs/tab_export.py
```

Do not redo this work in v6 unless further export-rendering polish is explicitly requested.

---

### Completed launcher isolation

The planned v6 clean launcher/release-folder work was pulled forward into v5.0.1.

The ship package now uses:

```text
SubtitleStudio_v5.0.1_ship_<timestamp>\
    Start Subtitle Studio.bat
    README_FIRST.txt
    app\
        SubtitleStudio.exe
        runtime files/folders
    _release_docs\
        release documentation
```

Important packaging rule preserved:

```text
Do not change Nuitka build layout.
Keep build_nuitka.ps1 authoritative.
Only make_ship_package.ps1 creates the isolated user-facing ship folder.
```

The momentary command prompt flash from the `.bat` launcher is accepted for v5.0.1 because it is still a major UX improvement over exposing the runtime file sea.

Future polish option:

```text
Replace BAT launcher with silent VBS or tiny compiled launcher.
```

Do not do this unless specifically requested.

---

### Packaged-app smoke evidence

The isolated package was tested successfully with runtime paths resolving under:

```text
release\...\app\bin\windows\ffmpeg.exe
release\...\app\bin\windows\ffprobe.exe
```

Confirmed from packaged app:

- MPV diagnostics OK
- MPV preview backend OK
- Video load OK
- Source audio extraction OK
- Waveform generation OK
- Premium pyrnnoise audio enhancement OK
- FFmpeg light polish OK
- Enhanced waveform reload OK
- IN/OUT marking OK
- Section creation OK

v6 should not redo these completed items unless further polish is requested.

---

## Product Themes

### 1. Core Edit Trust

Users must trust what they see while editing.

Includes:

- Subtitle preview should not vanish unexpectedly
- Subtitle preview should stay inside the actual video area
- Subtitle preview should be toggleable on/off
- Subtitle timing should be frame-aware
- Subtitle bin editing should remain smooth near the bottom of the list

---

### 2. Professional Editor Workflow

The Edit tab should feel closer to a real editing tool.

Includes:

- Frame-aware timecode editing
- Cleaner transport controls
- Existing Seek control renamed/repositioned as GO
- Existing frame-step/skip controls placed more clearly
- Existing Loop IN/OUT control placed more clearly
- Editable IN/OUT and section timecodes
- Play selected section
- Ghost-mark selected section on timeline
- Clearer playhead/waveform colours

---

### 3. Cleaner Workspace

The Edit tab should be less crowded and less technical.

Includes:

- Remove Resolution control from Edit tab
- Remove redundant Subtitle Font Size control from Edit tab
- Hide Log from default workspace tabs
- Open Log from menu when needed

---

### 4. Safer Operations

Long-running or cancellable operations should leave the app in a trustworthy state.

Includes:

- Cancel audio enhancement mid-process
- Fix US/UK conversion radio state after cancellation

---

### 5. Better Deliverables

Export outputs should match user expectations.

Includes:

- Replace DOCX-as-subtitle export with formatted Word transcript export

---

### 6. Cleaner Release Experience

Most of this work was completed early in v5.0.1.

Completed:

- Clean ship folder with obvious launcher
- Nuitka runtime folder kept intact under an `app` folder
- `Start Subtitle Studio.bat` added at package root
- `README_FIRST.txt` added at package root

Remaining optional future polish:

- Replace BAT launcher with silent VBS or tiny compiled launcher if specifically requested

---

## Confirmed Already Working / Not New v6 Work

These items were verified and should not be treated as new features:

- Add section
- Delete section
- Move section up
- Move section down
- Export order matches visible section order
- Keyboard shortcuts behave correctly
- Export works
- Existing Seek-to-time behavior exists
- Existing frame step / frame skip behavior exists
  - Shift currently skips 5 frames
- Existing Loop IN/OUT behavior exists
- Active app tabs are currently `Edit | Export | Log`
- Legacy Import/Review/Convert tabs were removed
- Legacy build scripts were retired
- Current authoritative build script is `build_nuitka.ps1`
- Burned subtitle smart curly quotes completed in v5.0.1
- Burned subtitle merged square-corner background shapes completed in v5.0.1
- Burned subtitle left/center/right alignment preserved after merged-shape fix
- Launcher isolation into `Start Subtitle Studio.bat` + `app\` folder completed in v5.0.1
- `make_ship_package.ps1` now creates the clean user-facing release folder

---

## Export Rendering Knowhow Preserved from v5.0.1

The burned subtitle renderer now uses custom ASS generation.

Important implementation model:

```text
Layer 0: merged vector background shape
Layer 1: positioned subtitle text
```

Do not return to ASS per-line boxed backgrounds using `BorderStyle=4`, because that caused visible dark overlap when transparent boxes touched or overlapped.

Current desired model:

```text
One subtitle cue
 -> split into visual lines
 -> measure/estimate each line width
 -> create padded rectangle per line
 -> merge rectangles into one square-corner vector shape
 -> draw background on lower ASS layer
 -> draw text lines above it
```

Brand shape behavior:

```text
Top line wider    -> T-shaped merged box
Bottom line wider -> inverted-T-shaped merged box
Equal widths      -> unified rectangle
```

Do not add rounded corners unless explicitly requested.

Quote handling:

```text
"Hello"  -> “Hello”
'Hello'  -> ‘Hello’
don't    -> don’t
John's   -> John’s
```

Smart quote conversion should remain export-only unless a future product decision explicitly says editor/source subtitle text should also be changed.

Known important functions in `tabs/tab_export.py`:

```text
_smart_curly_quotes
_ass_escape_text
_estimate_ass_text_width
_line_background_rect
_merged_rect_union_points
_ass_draw_path
_write_current_subs_to_temp
```

Risk note:

```text
Burned subtitle ASS rendering is visual-output-sensitive.
Any future changes must be tested with left, center, and right alignment,
single-line cues, two-line cues, quotes, apostrophes, and enhanced-audio video export.
```

---

## Roadmap Packages

---

## Package A — Frame / Timecode Foundation

### A1. Frame-aware timecode editing based on video FPS

Original proposal: Change 15

#### User value

Very high.

Users should be able to work in frame-aware timecodes after importing a video.

#### Desired behavior

On video import/load:

```text
Detect video FPS
Store FPS centrally
Use FPS for frame-aware editing, snapping, stepping, and validation
```

Recommended internal model:

```text
Keep seconds internally.
Add frame-aware conversion at UI boundaries.
```

Supported input should include:

```text
HH:MM:SS.mmm
HH:MM:SS,mmm
HH:MM:SS:FF when FPS is known
```

Recommended display for frame-aware edit controls:

```text
HH:MM:SS:FF
```

#### Important rule

Do not automatically snap imported subtitle timings in v6.

Imported subtitle timings should remain unchanged unless the user edits them.

#### Likely files

```text
core/timecode.py
core/ffmpeg.py
core/state.py
core/parser.py
tabs/tab_edit.py
tests/
```

#### Risk

Medium-high.

Timing is core product behavior.

#### Test plan

- Load video and confirm FPS is detected/logged
- Confirm frame-aware helper tests pass
- Confirm imported subtitle timings are preserved
- Confirm edited times snap to frame boundaries
- Confirm SRT/VTT export still writes valid millisecond timestamps
- Confirm existing tests pass

---

## Package B — Subtitle Preview Reliability

### B1. Fix subtitle preview disappearing during edit

Original proposal: Change 2

#### User value

Very high.

The subtitle preview must be reliable during editing.

#### Desired behavior

If Subtitle Preview is enabled and playhead is inside a cue range:

```text
Subtitle should be visible.
Edited text should appear after edit is saved.
Preview should not go blank unless no cue is active.
```

#### Likely files

```text
tabs/tab_edit.py
core/player_mpv.py
core/state.py
```

#### Risk

Medium.

Preview behavior intersects playback, cue selection, edit commit, and refresh timers.

#### Test plan

- Edit subtitle text while paused inside cue
- Confirm preview updates after save
- Select different cues
- Seek across cue boundaries
- Play through multiple cues
- Confirm preview only clears when outside cue timing

---

### B2. Keep subtitle preview inside actual video area

Original proposal: Change 17

#### User value

High.

Preview subtitles should not align to the edge of the preview window or black bars.

#### Desired behavior

Subtitle overlay should be positioned relative to the displayed video rectangle, not the outer preview widget.

Use safe margins:

```text
Horizontal margin: max(20 px, 5% of displayed video width)
Vertical margin: max(16 px, 5% of displayed video height)
```

#### Important rule

This affects preview only.  
It must not change burned subtitle export.

#### Likely files

```text
tabs/tab_edit.py
core/player_mpv.py
core/widgets.py
```

#### Risk

Medium.

Must handle resizing, aspect ratios, vertical video, long text, and alignment options.

#### Test plan

- Test left/center/right subtitle alignment
- Resize app window
- Test 16:9, 4:3, and vertical video if available
- Confirm export remains unchanged

---

### B3. Add Subtitle Preview On/Off option

Original proposal: Change 7

#### User value

High.

Users should be able to hide/show subtitle overlay during editing.

#### Desired behavior

Add a preference-backed toggle:

```text
Subtitle Preview: On / Off
```

Recommended menu placement:

```text
View > Show Subtitle Preview
```

Default:

```text
On
```

#### Important rule

This must only affect preview display.

It must not affect:

- subtitle data
- subtitle export
- burned subtitle export
- QC
- section export

#### Likely files

```text
main.py
tabs/tab_edit.py
core/state.py
```

#### Risk

Low-medium.

#### Test plan

- Toggle off while subtitle visible
- Confirm overlay disappears
- Toggle on while inside cue
- Confirm overlay reappears
- Edit while off
- Turn on and confirm edited text appears
- Restart app and confirm preference persists

---

## Package C — Subtitle Bin / Edit Tab UX

### C1. Make Find bar reactive while typing

Original proposal: Change 12

#### User value

High.

Find should respond as the user types.

#### Desired behavior

- Matches update after short debounce
- Match count updates
- Current match is selected/highlighted
- Enter goes to next match
- Shift+Enter goes to previous match if feasible
- Escape clears find if feasible

Recommended debounce:

```text
150-250 ms
```

#### Important rule

Do not filter/hide subtitle rows by default.

Highlight/select matches while preserving full context.

#### Likely files

```text
tabs/tab_edit.py
```

#### Risk

Medium-low.

---

### C2. Smooth scrolling/editing near bottom of Subtitle Bin

Original proposal: Change 3

#### User value

High.

Editing near the bottom should not jump, lose selection, or fight the user.

#### Desired behavior

- Edited row remains visible
- Selection remains stable
- Scroll position is preserved after edit
- Inline editor appears correctly
- Auto-follow does not fight active editing

#### Likely files

```text
tabs/tab_edit.py
```

#### Risk

Medium.

---

### C3. Remove Resolution control from Edit tab

Original proposal: Change 13

#### User value

Medium-high.

Resolution is an advanced display/performance setting and should not crowd the main Edit tab.

#### Desired behavior

Remove visible resolution control from Edit tab.

Keep preview resolution accessible from:

```text
View > Preview Resolution
```

#### Likely files

```text
tabs/tab_edit.py
main.py
```

#### Risk

Low-medium.

---

### C4. Remove redundant Subtitle Font Size option from Edit tab

Original proposal: Change 14

#### User value

Medium-high.

Font size is a display preference, not a primary editing action.

#### Desired behavior

Remove visible font size control from Edit tab.

Preserve default/stored behavior unless inspection confirms it is unused.

If needed later, expose through:

```text
View > Subtitle Preview Font Size
```

#### Likely files

```text
tabs/tab_edit.py
main.py
core/theme.py
```

#### Risk

Low-medium.

---

### C5. Reorganize Edit tab transport and action controls

Original proposal: Change 12

#### User value

Very high.

The Edit tab should be cleaner, less crowded, and more professional.

#### Important correction

These are already existing behaviors and should not be added as new features:

```text
Seek-to-time exists; rename/reposition as GO
Frame step / frame skip exists
Loop IN/OUT exists
```

#### Desired transport row

Use existing behavior where possible:

```text
[Time Entry] [GO] [Set IN] [Set OUT] [Go IN] [Go OUT]
[Step Back] [Play/Pause] [Stop] [Step Forward] [Loop IN/OUT]
[Volume] [Enhance Audio]
```

#### Desired subtitle action row

```text
[Save IN/OUT] [Clear IN/OUT]
[Sub Cue Start] [Sub Cue End] [Split Sub Cue]
[Add New Cue] [Delete Cue]
```

#### Recommended implementation rule

Move/relabel existing controls first.  
Do not add new playback behavior in the same commit.

#### Likely files

```text
tabs/tab_edit.py
```

#### Risk

Medium.

---

### C6. Make playhead and waveform visually distinct

Original proposal: Change 16

#### User value

Medium-high.

Timeline readability improves when waveform and playhead have clear visual hierarchy.

#### Desired behavior

Recommended direction:

```text
Waveform: muted gray
Playhead: bright accent / high contrast
```

Visual hierarchy:

```text
1. Playhead
2. IN/OUT markers
3. Selected section ghost range
4. Waveform
5. Background/grid
```

#### Likely files

```text
core/widgets.py
core/theme.py
tabs/tab_edit.py
```

#### Risk

Low-medium.

---

## Package D — Section Workflow

### D1. Make IN/OUT timecodes editable in preview section

Original proposal: Change 8

#### User value

Very high.

Users should be able to precisely correct IN/OUT values.

#### Desired behavior

Current IN/OUT display should support manual editing.

Validation:

- timecode must parse
- time must be >= 0
- time must be <= video duration if known
- IN must be before OUT
- OUT must be after IN

Use frame-aware parsing/snap once Package A exists.

#### Likely files

```text
tabs/tab_edit.py
core/timecode.py
core/state.py
```

#### Risk

Medium.

---

### D2. Make Sections Bin IN/OUT timecodes editable

Original proposal: Change 8

#### User value

Very high.

Saved sections should be precisely adjustable.

#### Desired behavior

Users can edit saved section IN and OUT values directly in the Sections Bin.

After edit:

- section row updates
- duration updates if shown
- export uses updated values
- selected-section ghost mark updates if present
- visible order remains unchanged

#### Likely files

```text
tabs/tab_edit.py
core/timecode.py
core/state.py
```

#### Risk

Medium.

---

### D3. Ghost-mark selected section on timeline

Original proposal: Change 6

#### User value

High.

Selecting a saved section should visually highlight its timeline range.

#### Desired behavior

Selecting a section shows a non-destructive ghost range.

Important:

```text
Ghost section range must not overwrite active IN/OUT marks.
```

#### Likely files

```text
tabs/tab_edit.py
core/widgets.py
```

#### Risk

Medium.

---

### D4. Playback selected section

Original proposal: Change 6

#### User value

High.

Users should be able to quickly review a saved section.

#### Desired behavior

```text
Select section
Click Play Section
Playback starts at section IN
Playback stops/pauses at section OUT
```

Recommended rule:

```text
Manual seek/pause cancels section playback mode.
```

#### Likely files

```text
tabs/tab_edit.py
core/player_mpv.py
```

#### Risk

Medium.

---

## Package E — State Safety / Cancel Behavior

### E1. Fix US/UK English conversion radio state after cancel

Original proposal: Change 4

#### User value

High.

Only one conversion direction should ever appear selected.

#### Desired behavior

- US and UK options are mutually exclusive
- Cancel restores previous valid selection
- Cancel does not leave both radio buttons active
- Conversion direction is clear before next operation

#### Likely files

```text
tabs/tab_edit.py
core/dictionary.py
core/state.py
```

#### Risk

Low-medium.

---

### E2. Cancel audio enhancement mid-process

Original proposal: Change 1

#### User value

High.

Users should be able to stop a long audio enhancement process.

#### Desired behavior

During enhancement:

```text
Enhance Audio changes to Cancel Enhancement
```

On cancel:

- running work stops safely
- FFmpeg subprocess is terminated if needed
- partial temp output is deleted or ignored
- app returns to normal UI state
- previous known-good enhanced audio is preserved if it existed
- source audio remains active if no previous enhanced audio existed

#### Likely files

```text
tabs/tab_edit.py
core/audio_enhance.py
core/ffmpeg.py
core/state.py
```

#### Risk

Medium.

---

## Package F — Export / Deliverables

### F1. Replace DOCX subtitle export with formatted Word transcript export

Original proposal: Change 9

#### User value

High.

A Word document should be a readable transcript, not a subtitle file format.

#### Desired behavior

Remove DOCX from normal subtitle format choices.

Add:

```text
Export Transcript (.docx)
```

Recommended first version:

- title
- source filename
- generated date/time
- one cleaned paragraph per subtitle cue
- timestamp prefix included by default

Example:

```text
[00:00:01.200] Welcome to the session.
```

#### Recommended implementation

Add a dedicated module:

```text
core/transcript_export.py
```

Use existing `python-docx` dependency.

#### Important note

Do not disturb the v5.0.1 burned subtitle ASS rendering hotfix when changing export UI or transcript export behavior.

#### Likely files

```text
tabs/tab_export.py
core/transcript_export.py
tests/
```

#### Risk

Medium-low.

---

## Package G — Log / Release UX

### G1. Hide Log from default tabs and open from menu

Original proposal: Change 11

#### User value

High.

Normal users do not need a permanent Log tab.

#### Desired behavior

Default main workspace:

```text
Edit | Export
```

Log available from:

```text
View > Open Log
```

Recommended implementation:

```text
Open Log in a separate window.
If already open, focus it.
Ctrl+3 opens/focuses Log window.
```

#### Important rule

Do not remove backend logging.

#### Likely files

```text
main.py
tabs/tab_logs.py
README.md
SMOKE_TEST_CHECKLIST.txt
SESSION_CONTEXT.txt
```

#### Risk

Medium.

---

### G2. Clean user-facing launcher/release folder

Original proposal: Change 5

#### Status

Completed early in v5.0.1.

Do not treat this as pending v6 work unless further launcher polish is requested.

#### User value

Very high for distribution.

Users should not need to find the executable inside runtime files.

#### Implemented release layout

```text
SubtitleStudio_v5.0.1_ship_<timestamp>\
    Start Subtitle Studio.bat
    README_FIRST.txt
    app\
        SubtitleStudio.exe
        runtime files/folders
    _release_docs\
        release documentation
```

#### Important rule

Do not change Nuitka build layout.

Keep `build_nuitka.ps1` stable.

Modify ship package process only:

```text
make_ship_package.ps1
```

#### Future optional polish

```text
Replace Start Subtitle Studio.bat with a silent launcher.
```

Do not do this unless specifically requested.

#### Files already touched

```text
make_ship_package.ps1
```

#### Risk if revisited

Low-medium.

---

## Recommended Implementation Order

This is the recommended v6 development order after v5.0.1 completed work.

```text
1. Frame-aware timecode foundation
2. Fix subtitle preview disappearing during edit
3. Keep subtitle preview inside video area
4. Add Subtitle Preview On/Off toggle
5. Reactive Find bar
6. Smooth Subtitle Bin scrolling/editing near bottom
7. Remove Resolution control from Edit tab
8. Remove Subtitle Font Size option from Edit tab
9. Reorganize existing transport/action controls
10. Editable current IN/OUT timecodes
11. Editable Sections Bin IN/OUT timecodes
12. Ghost-mark selected section on timeline
13. Play selected section
14. Make playhead and waveform visually distinct
15. Fix US/UK radio state after cancel
16. Cancel audio enhancement mid-process
17. Replace DOCX subtitle export with Word transcript export
18. Hide Log from default tabs and open from menu
19. Clean user-facing launcher/release folder — completed early in v5.0.1
```

---

## Development Rules for v6

1. No broad rewrites.
2. One small change at a time.
3. Prefer one commit per stable unit.
4. Run compile checks after each touched Python file.
5. Run `python run_tests.py` after each meaningful feature.
6. Do not touch `build_nuitka.ps1` unless packaging QA exposes an issue.
7. Do not change export behavior while fixing preview behavior.
8. Do not silently alter imported subtitle timings.
9. Keep seconds internally; use frame conversion at UI boundaries.
10. Preserve the v5.0.1 burned subtitle ASS rendering model unless specifically asked to change it.
11. Update `SESSION_CONTEXT.txt` and regenerate clean export at handoff.

---

## Standard Test Gate

Use as applicable:

```powershell
python -m py_compile main.py
python -m py_compile tabs\tab_edit.py
python -m py_compile tabs\tab_export.py
python -m py_compile tabs\tab_logs.py
python -m py_compile core\ffmpeg.py
python -m py_compile core\state.py
python -m py_compile core\parser.py
python run_tests.py
```

Only compile files that exist and/or were touched.

---

## Burned Subtitle Rendering Regression Test

Because v5.0.1 fixed visual burned subtitle rendering, any future export-related work should test:

```text
"Hello," she said.
'Hello,' he replied.
It's John's file.
He said, "It's called 'Subtitle Studio.'"

"This is the wider top line"
short bottom

short top
"This is the much wider bottom line"

"First quoted line"
"Second quoted line"
```

Expected:

```text
“Hello,” she said.
‘Hello,’ he replied.
It’s John’s file.
He said, “It’s called ‘Subtitle Studio.’”
```

Visual expectations:

```text
No overlapping transparent subtitle boxes.
Two-line subtitles use one square-corner merged stepped background shape.
Top wider line creates T-shaped box.
Bottom wider line creates inverted-T-shaped box.
Text remains vertically centered.
Padding remains visually uniform.
Left, center, and right alignment work.
```

---

## Handoff Requirements

At session end, update/regenerate:

```text
SESSION_CONTEXT.txt
PROJECT_CODE_EXPORT_CLEAN_MANIFEST.txt
PROJECT_CODE_EXPORT_CLEAN_PART_001.txt
V6_ROADMAP.md
```

If the export manifest reports more parts, include all parts in order.

---

## Current Next Step

Before coding v6 Package A, inspect the current code paths for frame/time handling:

```text
core/ffmpeg.py
core/state.py
core/parser.py
tabs/tab_edit.py
```

Goal:

```text
Map how video FPS is currently probed/stored
Map how timecodes are parsed/formatted
Map where user time edits are applied
Design exact minimal implementation plan for frame-aware timecode support
```

No v6 Package A code should be changed until the exact implementation map is approved.
