# Why Subtitle Studio Exists

I spent 17 years in broadcast motion design and video editing.  
Every day, I saw the same friction:

- Editors manually burning subtitles frame by frame  
- Copy‑pasting text between Word and SRT files  
- Audio enhancement requiring expensive plugins or cloud APIs  
- Exporting clips meant typing complex FFmpeg commands  

**Existing tools failed in three ways:**

1. **Cloud‑dependent** – useless in air‑gapped enterprise environments  
2. **Expensive** – monthly fees for basic subtitle styling  
3. **Fragmented** – one app for editing, another for burning, another for audio  

Subtitle Studio is my answer.  
It’s a **single, offline‑first, zero‑install executable** that handles:

- Video preview (MPV)  
- Subtitle editing (inline, spell check, find/replace, alignment)  
- Clip selection (IN/OUT, reorder, delete)  
- Premium audio enhancement (local pyrnnoise)  
- Export: video (burned subtitles, resolution, compression), audio (MP3/AAC/WAV), subtitles (SRT/VTT/TXT/DOCX/FCPXML)  

---

## The Philosophy

**“Precision first. Preview trust second. Workflow polish third. Packaging last.”**

- **Precision** – frame‑accurate timing, smart quote conversion, merged subtitle backgrounds  
- **Trust** – subtitle preview must never vanish; waveform must match audible audio  
- **Polish** – remove clutter (Resolution, Font Size from Edit tab), make export intuitive  
- **Packaging** – Nuitka + manual DLL copies = one `app` folder + launcher BAT  

---

## What Makes It Different from Tutorial Projects

- **It ships.** I’ve compiled and distributed v5.0.1 to internal users.  
- **It has a roadmap.** v6 is planned with concrete packages (frame‑aware timecode, cancel enhancement, etc.).  
- **It has release documentation.** Smoke tests, runtime manifests, build scripts, regression cases.  
- **It solves a real workflow.** Not a demo – a tool that replaces “Shadow IT” converters and manual subtitle burns.  

---

## Who This Is For

- Recruiters and hiring managers looking for **system architects** who can own a product end‑to‑end.  
- Engineering leads who value **release discipline** and **technical communication**.  
- Teams building **AI‑native desktop tools** – my pyrnnoise integration and LLM‑assisted development are directly transferable.  

---

## How to Read This Repository

| File | What It Shows |
|------|----------------|
| `V6_ROADMAP.md` | Product thinking, prioritisation, risk management |
| `DEV_JOURNAL.md` | Engineering decisions, packaging war stories, release process |
| `PROJECT_CODE_EXPORT_CLEAN_PART_001.txt` | Actual Python source (53 files, organised) |
| `subtitle_studio_ui_preview.png` | Visual proof of the Edit tab |
| `PROJECT_CODE_EXPORT_CLEAN_MANIFEST.txt` | Complete file list |

---

*Built with Python, Nuitka, FFmpeg, MPV, pyrnnoise, and a lot of late nights.*  
*— Aniket Mukherjee*
