# Subtitle Studio – Development Journal (Engineering Summary)

**Project:** Subtitle Studio v5 → v6  
**Role:** Solo architect & developer  
**Timeline highlight:** v5 built in 30 days (concept → compiled ship package)

---

## Core Problem Solved

Professional video editors waste hours on repetitive subtitle tasks:  
- Burning subtitles with brand styling  
- Aligning multi‑line quotes  
- Switching between source and enhanced audio  
- Exporting clips, trimmed videos, and sidecar files  

Existing tools are either cloud‑dependent, expensive, or require complex FFmpeg commands.

**Subtitle Studio is an air‑gapped, zero‑dependency desktop app** that puts a professional subtitle workflow inside a single executable.

---

## Key Technical Decisions

### 1. Standalone & Air‑Gapped
- Compiled with Nuitka (no Python install required)  
- All dependencies bundled: FFmpeg, MPV, PyAV, pyrnnoise, NumPy, SoundFile  
- Works on restricted enterprise networks, no internet access needed  

### 2. Premium Audio Enhancement
- Integrated `pyrnnoise` (RNNoise) for local speech denoising  
- Replaced cloud transcription vendors with local AI vocal isolation  
- Fallback to FFmpeg `arnndn` when pyrnnoise unavailable  

### 3. MPV Preview Backend
- Replaced OpenCV frame‑buffer with `libmpv` for smooth A/V sync  
- Supports external audio switching (source ↔ enhanced) without reloading video  
- Handles corrupt and silent videos gracefully  

### 4. Burned Subtitle Rendering (Custom ASS Generator)
- Merged multi‑line subtitle backgrounds into **one square‑corner shape** (no overlapping transparent boxes)  
- T‑shaped / inverted‑T‑shaped boxes depending on line widths  
- Smart curly quotes (export‑only, does not mutate source text)  
- Left / center / right alignment preserved  

### 5. Export Engine (v5 Plan Model)
- Multi‑output: video + audio + subtitles in one export  
- Selection scopes: full source, remove clip selections, separate clips, merged clips  
- Subtitle multi‑format: SRT, VTT, TXT, DOCX, FCPXML  
- Audio formats: MP3, AAC, WAV (with bitrate control)  
- Compression presets + custom target size (two‑pass encoding ready)

---

## Release Discipline

- **Smoke test checklist** (SMOKE_TEST_CHECKLIST.txt)  
- **Runtime manifest** (RUNTIME_MANIFEST_SHIP.txt) – exactly which DLLs and packages are bundled  
- **Build script** (`build_nuitka.ps1`) with manual copies for PyAV, audiolab, pyrnnoise – because automatic dependency resolution failed  
- **Ship package creator** (`make_ship_package.ps1`) – isolates the executable and runtime into a clean user‑friendly folder  

Key packaging rules preserved across versions:


---

## What v6 Will Bring (in progress)

- Frame‑aware timecode editing (HH:MM:SS:FF)  
- Subtitle preview reliability (never vanishes during edit)  
- Cancel audio enhancement mid‑process  
- Editable IN/OUT and section timecodes  
- Play selected section + ghost mark on timeline  
- Cleaner Edit tab layout (remove resolution/font size clutter)  
- Log moved to a separate window (default workspace: Edit | Export)

---

## Why This Project Matters for My Roles

| Role Requirement | How Subtitle Studio Demonstrates It |
|----------------|--------------------------------------|
| End‑to‑end product ownership | I built it alone – from idea to compiled Windows package. |
| AI‑native systems | Local pyrnnoise inference, LLM‑assisted development (Claude/Gemini). |
| Systems thinking | Roadmap, packaging constraints, regression testing, release documentation. |
| Operational excellence | Smoke tests, runtime manifests, build automation, clean‑machine QA. |
| Human‑in‑the‑loop | Preview overlay, alignment controls, undo stack, pre‑export QC. |
| Standalone deployment | Air‑gapped, no cloud, no Python install. |

---

## Source Code Access

The full source code is available in this repository:  
`PROJECT_CODE_EXPORT_CLEAN_PART_001.txt` (53 files, ~1.5 MB).  

Key modules:
- `core/audio_enhance.py` – hybrid pyrnnoise + FFmpeg pipeline  
- `core/player_mpv.py` – MPV integration  
- `tabs/tab_edit.py` – subtitle bin, inline editing, spell check, sections  
- `tabs/tab_export.py` – ASS burner, multi‑output export planner  
- `core/state.py` – undo, audio path management, project save/load  

---

*Last updated: May 2026*  
*Part of the public engineering portfolio for Aniket Mukherjee*
