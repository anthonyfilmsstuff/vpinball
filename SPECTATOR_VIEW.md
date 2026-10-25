# Spectator View for Visual Pinball X 🎥🎰

Hey — I'm Anthony ([@anthonyfilmsstuff](https://github.com/anthonyfilmsstuff)). This is a fork of
[Visual Pinball X](https://github.com/vpinball/vpinball) that adds something I really wanted and
couldn't find: a **second, steady camera view of the playfield** that runs *at the same time* as your
normal game.

The itch I was scratching: I stream pinball in VR, and the head-tracked view is great to *play* but
rough to *watch* — every little head movement makes viewers seasick. So now I can keep my VR view in
the headset while a calm, fixed "broadcast" shot of the table goes to a separate window I point OBS at.
Works on a flat desktop too (you get a dedicated spectator window), and the DMD/score shows up in it.

> ### 🛠️ Heads up: this is a vibe-coded passion project
> Full transparency — **I'm not a C++ developer.** I built this by pairing with Claude (Anthropic's
> AI), one step at a time, driven purely by wanting this feature to exist. It's been tested and it
> works really well for me, but it's a labor of love, not a battle-hardened engine contribution.
> **Expect some rough edges**, and if you're an actual graphics/engine dev, I would *love* the help
> making it better. 🙏

The virtual pinball community on Reddit got me way more excited about this than I expected, so here it
is — come kick the tires.

---

## ✨ What it does

- **A steady, independent camera** — a fixed third-person shot that's totally separate from your
  head/cabinet view.
- **VR and desktop** — in VR it takes over the preview window; on desktop it opens its own window.
- **Split view (VR)** — show your **first-person eye on the left and the steady camera on the right**,
  in one window. Great for streams that want both.
- **Three camera angles** — Angled, Overhead, Low — flip between them live.
- **Frame it however you like** — pan, tilt, zoom, and move the camera closer/farther on the fly.
- **It remembers per table** — set up a shot, save it, and that table loads with your shot next time.
- **A throttle** — render the spectator every frame, or every 2nd/3rd, so your headset always gets
  priority.

---

## 🎮 Controls

Everything's live while a table is running. (Printable/Discord-friendly version in
[`CONTROLS.md`](CONTROLS.md).)

### View modes
| Key | What it does |
|-----|--------------|
| **F7** | Toggle the spectator view (in VR: spectator ⇄ mirrored eye) |
| **F5** | Split view (VR): mirrored eye **left** + spectator camera **right** |
| **F6** | Cycle the update rate — every frame → 2nd → 3rd |

### Perspective & view — frame the shot
These are the keys that actually move and aim the camera:

| Key | What it does |
|-----|--------------|
| **F8** | Cycle the camera angle/preset — Angled → Overhead → Low |
| **↑ ↓ ← →** | Pan the camera (up / down / left / right) |
| **`[` / `]`** | Tilt the angle (shallower / steeper) |
| **KP `+` / `−`** | Zoom in / out (field of view) |
| **`,` / `.`** | Move the camera closer / farther (distance/dolly) |
| **F4** | Reset the perspective back to the current preset |

### Save
| Key | What it does |
|-----|--------------|
| **F9** | Save the camera for this table (auto-loads next time you play it) |

> The perspective/framing keys are unbound by default in VPX, so they shouldn't fight your gameplay
> keys. If you *have* rebound any of them, all the new actions show up in VPX's key settings under
> **"Spectator …"**.

---

## 🚀 Trying it out (testers, this is you)

> ⚠️ **This is an experimental build — please keep your normal Visual Pinball install separate and
> don't overwrite it.** Run this one on its own.

1. Grab the build (the whole folder — the `.exe` needs its DLLs and `assets` next to it).
2. Unzip it somewhere by itself.
3. Windows will probably throw a **"unknown publisher"** warning, because I haven't code-signed it —
   that's normal for a community build. **More info → Run anyway** if you trust it.
4. Fire up a table (double-click the exe to browse, or launch straight in):
   ```
   VPinballX_BGFX64.exe -play "C:\path\to\your\table.vpx"
   ```
5. Frame your shot with the keys above, hit **F9** to save it for that table, and you're set.

**OBS:** add a **Window Capture** and grab the **`Visual Pinball VR Preview`** window (or
**`Visual Pinball Spectator`** on desktop). If it's black, switch the capture method to **Windows
Graphics Capture (WGC)**. If you're on an AMD card like me, use the GPU's hardware encoder (AMF, AV1 is
great) so encoding doesn't eat your frame budget.

---

## 💾 Per-table cameras

Hit **F9** and I save your camera to a little text file called `<your-table>.vpx.spectatorcam`, right
next to that table's `.vpx`. Next time you play it, your shot loads automatically. Every table keeps
its own angle (they all need different framing). Delete the file to forget it.

---

## ⚙️ Performance & recommended hardware

The spectator is a **second render of the table** each frame — so it costs real GPU, though it skips the
heavy bloom/reflection passes and renders at the preview resolution. Ballpark when it's running every
frame: **roughly +20–40% GPU in VR** (more on desktop), tiny on CPU/VRAM. The **F6 throttle** scales
that down, and your headset always comes first.

Rough tiers (it rides on top of whatever you already need for VPX VR):

| Tier | GPU | Notes |
|------|-----|-------|
| **Minimum** | RTX 3060 Ti / RX 6700 XT (8 GB) | Spectator **throttled** (F6), ~720p, lighter tables |
| **Recommended** | RTX 4070 / RX 7800 XT (12 GB) | Every-frame, 1080p, most tables |
| **High-end** | RTX 4080/4090 or RX 7900 XT/XTX (16–24 GB) | Every-frame on heavy tables *while streaming* |

For reference, I run it on a **Ryzen 9 7950X3D / RX 7900 XTX / 64 GB** and it eats everything I throw at
it with room to spare for OBS.

---

## 🔧 Building it yourself

Targets the **BGFX backend + OpenXR**, built on **Windows x64**. (See upstream
[`make/README.md`](make/README.md) for the full dependency setup — mine was the standard VS 2022
"Desktop development with C++" workload plus the prebuilt third-party deps.)

Quick version:
1. Get the prebuilt third-party deps into `third-party/` (the `*-third-party-windows-x64-Release.zip`
   artifact from a recent [vpinball Actions run](https://github.com/vpinball/vpinball/actions)).
2. Generate the VS solution: run `make/create_vs_solution.bat`, pick **2022**.
3. Build the **`vpx`** project, **`Release_BGFX` / `x64`**.
4. Output lands at `.build/bin/vpx/Release_BGFX-x64/VPinballX_BGFX64.exe`.

---

## 🔍 How it works (the short, honest version)

Vanilla VPX draws the table **once** a frame from your camera. This fork hooks into that same flow and,
after your main view is drawn, sets up a **separate camera**, **draws the table a second time** into its
own buffer, and shows it in another window. It reuses VPX's existing camera math, GPU layer, and headset
integration — the whole thing is about **~500 lines across 8 files**. The fiddly VR bits (keeping the
fixed camera from snapping to your head, and getting the image to actually show in the OpenXR preview
window) were most of the adventure.

Files touched: `plugins/plugins/VPXPlugin.h`, `src/renderer/{Renderer,Window}.{cpp,h}`,
`src/core/player.{cpp,h}`, `src/input/InputManager.cpp`.

---

## 📋 Status & known limits

- Built and tested on **BGFX + OpenXR + Windows x64**. Other backends (OpenGL, DX9) and platforms
  (macOS, Linux, standalone/mobile) aren't wired up.
- Desktop spectator window is for **mono** (non-stereo) sessions.
- It's an **experimental community build**, not an official VPX release.
- Again: **vibe-coded, rough edges likely.** That's where you come in. 😄

---

## 🤝 Want to help?

Testing, odd framing on specific tables, performance numbers — all super welcome. Please include your
**table, VR headset, what you did, what happened, and a screenshot or short clip.**

If you write code, some things I'd genuinely love help with: reusing cached static geometry to cut the
GPU cost, an in-game settings panel for the camera, more split layouts, and support for the other
backends/platforms.

---

## 📜 License & credits

This is a fork of Visual Pinball X, under the **GPLv3**, same as upstream — see [`LICENSE`](LICENSE).
Huge thanks to the [Visual Pinball X](https://github.com/vpinball/vpinball) team; this is built entirely
on top of their incredible work. I just added one camera. 🙂

— Anthony ([@anthonyfilmsstuff](https://github.com/anthonyfilmsstuff))
