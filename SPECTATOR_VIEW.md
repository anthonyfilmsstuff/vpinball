# Visual Pinball X — Third-Person Spectator View

> A fork of [Visual Pinball X](https://github.com/vpinball/vpinball) that adds a **second, fixed-camera
> live view of the playfield** — a stable "broadcast" shot you can stream or show to bystanders while
> you play, instead of the nauseating head-tracked view in VR.

The player keeps their normal view (VR headset or the cabinet playfield window). At the same time, a
**separate, steady camera** renders the same live game to its own window — perfect for streaming to OBS
or for people watching over your shoulder. The DMD/score is visible in the spectator view too.

---

## 📺 Demo

*(add a screenshot / short clip here — a side-by-side of your headset view and the spectator window is ideal)*

---

## ✨ Features

- **Independent fixed camera** — a stable third-person shot, completely separate from the player's
  head/cabinet camera.
- **Works in VR and on desktop**
  - **VR:** renders into the existing VR preview window (replacing the mirrored eye).
  - **Desktop:** opens its own dedicated spectator window.
- **Split mode (VR):** show the **mirrored first-person eye on the left, beside the steady camera on
  the right**, in one window — great for streams that want both.
- **Three camera presets** — Angled, Overhead, Low — cycle live.
- **Full live framing** — pan, tilt, zoom (FOV), and camera distance, adjusted on the fly.
- **Per-table memory** — save a camera per table; it auto-loads next time you play that table.
- **Performance throttle** — render the spectator every frame, every 2nd, or every 3rd frame, so the
  player's view always takes priority.

---

## 🎮 Controls

All keys are active while a table is running and the spectator view is available. See
[`CONTROLS.md`](CONTROLS.md) for a printable/Discord-pasteable version.

| Key | Action |
|-----|--------|
| **F7** | Toggle the spectator view (in VR: spectator ⇄ mirrored eye) |
| **F5** | Split mode (VR): mirrored eye **left** + spectator camera **right** |
| **F8** | Cycle camera preset — Angled → Overhead → Low |
| **F6** | Cycle update rate — every frame → every 2nd → every 3rd |
| **F4** | Reset camera to the current preset |
| **F9** | **Save** the camera for the current table |
| **↑ ↓ ← →** | Pan the camera (up / down / left / right) |
| **`[` / `]`** | Tilt the camera angle (shallower / steeper) |
| **KP `+` / `−`** | Zoom in / out (field of view) |
| **`,` / `.`** | Camera distance — move closer / farther |

> The adjust keys (arrows, `[ ]`, `, .`, numpad `+ −`) are unbound by default in VPX, so they should not
> clash with gameplay. If you've custom-bound any of them, all the new actions are rebindable in VPX's
> key settings under the **"Spectator …"** entries.

---

## 🚀 Quick start (for testers)

> ⚠️ **Experimental build.** Keep your existing Visual Pinball install intact and run this one
> **separately** — don't overwrite your normal setup.

1. Download the build (the whole folder — the `.exe` needs its DLLs and `assets` to run).
2. Unzip it somewhere of its own.
3. Windows may show a **"Windows protected your PC / unknown publisher"** warning, because this build
   isn't code-signed. That's expected for a community build — choose **More info → Run anyway** if you
   trust the source.
4. Launch a table:
   ```
   VPinballX_BGFX64.exe -play "C:\path\to\your\table.vpx"
   ```
5. Press the spectator keys above to frame it, then **F9** to remember it for that table.

**Capturing in OBS:** add a **Window Capture** source and pick the **`Visual Pinball VR Preview`** window
(or the **`Visual Pinball Spectator`** window on desktop). If the capture is black, switch the source's
**Capture Method** to **Windows Graphics Capture (WGC)**.

---

## 💾 Per-table cameras

When you press **F9**, the current camera is saved to a small text file named
`<your-table>.vpx.spectatorcam`, **right next to that table's `.vpx`**. The next time you play that
table, it loads automatically. Each table remembers its own shot; delete the file to forget it.

---

## ⚙️ Performance

The spectator is a **second render of the scene** each frame (it deliberately skips the heavy bloom /
ambient-occlusion / reflection passes and renders at the preview window's resolution). Rough cost when
running **every frame**: on the order of **+20–40% GPU in VR**, more on desktop, negligible CPU and a few
MB of video memory. The exact figure depends on the table and your resolution.

- Use **F6** to throttle (every 2nd/3rd frame) on demanding tables — it scales the cost down roughly
  linearly.
- To measure it on your own hardware: watch your frame time/GPU use and toggle the spectator with **F7**.

---

## 🛠️ Building from source

This fork targets the **BGFX backend with OpenXR**, built on **Windows x64**. (The change is small and
self-contained; see *How it works* below.)

**Prerequisites**
- Windows, [Git](https://git-scm.com/), and **Visual Studio 2022** with the **"Desktop development with
  C++"** workload.
- The prebuilt third-party dependencies (bgfx, SDL3, FreeImage, FFmpeg, OpenXR, etc.).

**Steps** (summary — see upstream [`make/README.md`](make/README.md) for full details)
1. Get the prebuilt third-party deps and extract them into `third-party/` (download the
   `*-third-party-windows-x64-Release.zip` artifact from a recent
   [vpinball Actions run](https://github.com/vpinball/vpinball/actions)).
2. Generate the Visual Studio solution: run `make/create_vs_solution.bat` and choose **2022**
   (it writes the solution to `.build/vsproject/`).
3. Build the **`vpx`** project, configuration **`Release_BGFX`**, platform **`x64`** (Visual Studio, or
   MSBuild from a developer prompt).
4. The result is `.build/bin/vpx/Release_BGFX-x64/VPinballX_BGFX64.exe`.

---

## 🔍 How it works (short version)

Vanilla VPX draws the table **once** per frame from the player's camera. This fork taps into that same
pipeline and, after the main view is drawn, sets up an **independent camera**, **draws the same parts a
second time** into its own buffer, tone-maps it, and presents it to a second window. It reuses VPX's
existing camera math, GPU layer, headset integration, and table parts untouched — the whole feature is
~500 lines across 8 files.

The two non-obvious VR details that took the most work: VPX normally re-derives the view from the
**headset pose** per object (so the fixed camera has to explicitly opt out of that), and the OpenXR
preview window only accepts an image drawn **in-frame via a textured-quad draw** (a direct buffer copy
doesn't get presented).

Files touched: `plugins/plugins/VPXPlugin.h`, `src/renderer/{Renderer,Window}.{cpp,h}`,
`src/core/player.{cpp,h}`, `src/input/InputManager.cpp`.

---

## 📋 Status & known limitations

- Built and tested on the **BGFX + OpenXR + Windows x64** path. Other backends (OpenGL, DX9) and
  platforms (macOS, Linux, standalone/mobile) are **not** wired up.
- Desktop spectator window is enabled for **mono** (non-stereo) sessions; 3D-TV/anaglyph desktop stereo
  is not covered.
- The mirrored-eye half of split mode is letterboxed to preserve aspect; the spectator half fills its
  panel.
- This is an **experimental community build**, not an official VPX release.

---

## 🤝 Contributing & feedback

Bug reports, tables that frame oddly, and performance numbers are all very welcome — please include your
**table name, VR headset/runtime, what you did, what happened, and a screenshot or short clip**.

Ideas for contributors (help wanted): reuse the cached static geometry for the spectator to cut GPU
cost, an in-game settings panel for the camera, more split layouts (top/bottom, swap sides), and
extending support to other backends/platforms.

---

## 📜 License

This is a fork of Visual Pinball X and is licensed under the **GPLv3**, the same as upstream. See
[`LICENSE`](LICENSE). The corresponding source is this repository.

## 🙏 Credits

- The [Visual Pinball X](https://github.com/vpinball/vpinball) project and its contributors — this builds
  directly on their work.
- Spectator-view feature: *(your name/handle here)*.
