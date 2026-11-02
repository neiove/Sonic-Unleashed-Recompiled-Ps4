<h1 align="center">Unleashed Recompiled — PS4</h1>

<p align="center">
  An unofficial PlayStation 4 port of <a href="https://github.com/hedge-dev/UnleashedRecomp">Unleashed Recompiled</a>,
  the static recompilation of the Xbox 360 version of <i>Sonic Unleashed</i>.
</p>

<p align="center">
  <img alt="Platform" src="https://img.shields.io/badge/platform-PS4%20(homebrew)-003791">
  <img alt="Status" src="https://img.shields.io/badge/status-playable%20%2F%20experimental-orange">
  <img alt="Version" src="https://img.shields.io/badge/version-0.84-blue">
</p>

> [!IMPORTANT]
> This project does **not** include any game assets. You must provide your own legally obtained copy of the
> Xbox 360 version of *Sonic Unleashed* and its title update. Running it requires a PS4 that can run homebrew.

> [!WARNING]
> This is an experimental port. It plays the full story, but expect bugs, missing effects in places you haven't
> recorded shaders for yet (see [Shaders](#shaders)), and the occasional crash.

---

## Table of contents
- [What this is](#what-this-is)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [The in-game PS4 menu (L3 + R3)](#the-in-game-ps4-menu-l3--r3)
- [Mods](#mods)
- [Performance](#performance)
- [Shaders](#shaders)
- [Files and folders on the console](#files-and-folders-on-the-console)
- [Troubleshooting](#troubleshooting)
- [Building from source](#building-from-source)
- [How it works](#how-it-works)
- [Credits](#credits)
- [Legal](#legal)

## What this is

Unleashed Recompiled turns the Xbox 360 game's PowerPC code into C++ that compiles natively for other platforms.
This repository takes that work to the PS4: the recompiled game code is built with the
[OpenOrbis](https://github.com/OpenOrbis/OpenOrbis-PS4-Toolchain) toolchain, and everything the PC version gets
from Windows, SDL and Vulkan is replaced with PS4-native code, including a **custom GPU backend that drives the
PS4's graphics hardware directly**.

## Features

### Gameplay
- 🎮 **Full game, natively on PS4.** No emulation: the game's code runs as native x86-64 on the console.
- 🌍 Day and night stages, hub worlds, cutscenes and the full story have been played through.
- 🕹️ **DualShock 4 support** read directly through the PS4 pad library.
- 📦 **Easy install:** drop the game, its title update and any DLC into one folder and the app installs them
  itself on first launch.
- 💾 Saves, settings and logs live in `/data/UnleashedRecomp`, so they survive reinstalling the PKG.

### Graphics
- 🖥️ **Native PS4 GPU renderer** (`plume_ps4`): hand-built command buffers submitted straight to the GPU, no
  OpenGL or Vulkan layer.
- 🧱 Proper tiled textures and render targets, compressed (BC) textures, cube maps, depth buffers and shadows.
- 🎨 **Pre-compiled shader pack** with 1500+ pipelines, built on a PC for the PS4's GPU generation.
- 🔁 Hardware primitive restart, depth bias, depth clip and GPU-side clears.
- 🛡️ **Hang protection:** if a shader ever hangs the GPU, the app remembers that pipeline and skips it next time
  instead of freezing again.

### Frame generation (experimental)
- ⚡ Turns a steady **30 fps into 60 fps on screen** by generating an in-between frame.
- Two modes: **Interpolate** (default, smoother) and **Extrapolate** (less latency).
- Uses the game's own motion vectors for characters and **camera reprojection from the depth buffer** for the
  world, with the HUD kept sharp and unmoved.
- Options to limit motion to the camera or Sonic, flip direction, or turn the HUD fix off, for tuning.
- Perfect frame pacing with a 4-buffer swapchain.

### Performance options
- 🌑 **Shadows on/off** live (the biggest single performance win).
- 📏 **Render distance:** Full / Far / Medium / Near.
- ✨ **Effects:** Full / Reduced / Off.
- 🎚️ **Frame rate cap:** 20 / 30 / 40 / 60 / unlimited.
- Resolution scale, texture quality, anisotropic filtering, shadow resolution, motion blur, VSync.
- 📊 **On-screen FPS counter.**

### Mods and cheats
- 🧩 **Mod support** using the same mod format as the PC version (`mod.ini`, Hedge Mod Manager-style mods).
- In-game **mod manager**: enable, disable and reorder mods without a PC.
- Handles mods made on Windows (case-insensitive file lookup on the PS4's case-sensitive file system).
- 🎟️ **Built-in codes / cheats** toggled from the Gameplay tab.

### PS4-specific tuning
- Works great on **PS4 Pro with Boost Mode** (CPU ~2.1 GHz is detected and logged).
- Custom thread waits, thread priorities and timers tuned for the PS4's kernel (these took gameplay from under
  10 fps to 30-60 fps).
- Detailed logs, crash reports and an optional sampling profiler for anyone who wants to help debug.

## Requirements

- A PS4 or PS4 Pro on a firmware that can run homebrew (for example with GoldHEN).
- A way to copy files to the console (FTP is easiest).
- **Your own copy** of the Xbox 360 version of *Sonic Unleashed* (as an `.iso` or extracted files).
- The game's **Xbox 360 title update** (required).
- Any DLC you own (optional).

## Installation

1. Download `Unleashed-Recomp.pkg` from [Releases](../../releases) and install it with your homebrew package
   installer (Debug Settings → Package Installer, or GoldHEN's installer).
2. Launch the app once. It creates `/data/UnleashedRecomp/` and an `Install` folder inside it, then tells you
   the game is missing.
3. Over FTP, copy into `/data/UnleashedRecomp/Install/`:
   - the game `.iso` (or its extracted folder),
   - the title update,
   - any DLC.
4. Launch the app again. It works out which file is which, checks the update matches the game, and installs
   everything into `/data/UnleashedRecomp`. This only happens once; you can delete the `Install` folder after.
5. Play!

## The in-game PS4 menu (L3 + R3)

Press **L3 + R3** together at any time to open the PS4 menu. The game keeps running behind it.

| Tab | What's in it |
| --- | --- |
| **Performance** | Shadows, frame rate cap, frame generation, render distance, effects, resolution scale, texture quality, anisotropy, shadow resolution, motion blur, VSync, show FPS |
| **Gameplay** | Built-in codes / cheats |
| **Mods** | Every mod in the Mods folder: **✕** toggles, **□ / △** reorder |
| **System** | Restart the game (saves first) |

Settings are saved to `/data/UnleashedRecomp/ps4_menu.ini` and `config.toml`.

## Mods

1. Copy each mod's folder (the one containing `mod.ini`) into `/data/UnleashedRecomp/Mods/`.
2. Open the PS4 menu (L3 + R3) → **Mods**, enable the mods you want and set their order.
3. Use **System → Restart** (or relaunch the app). Mods apply from the next launch.

> [!NOTE]
> Mods that add **new models or materials** may show up invisible the first time, because the PS4 can't compile
> shaders by itself. Those need their shaders recorded and added to the shader pack (see below). Texture, text
> and file-replacement mods work straight away.

## Performance

Rough numbers on a PS4 Pro with Boost Mode:

| Where | FPS |
| --- | --- |
| Title screen | 60 |
| Hub worlds | ~30-45 |
| Stages | ~25-45 (higher with shadows off) |
| With frame generation | 60 on screen (30 real + 30 generated) |

The game is mostly **CPU-bound** on PS4 (the GPU needs only ~5 ms a frame). For the smoothest experience:
**turn shadows off, cap at 30 fps, and turn on frame generation.**

## Shaders

The PS4 can't compile shaders at runtime, so every shader combination the game uses is compiled ahead of time on
a PC and shipped in `ps4_shaders.pack` inside the PKG.

- If you reach a place whose shaders aren't in the pack, those objects are **skipped** (they'll be missing on
  screen) and the app **records** what it needed to `/data/UnleashedRecomp/ps4_record/`.
- Send those recordings in (open an issue) and they get compiled into the next pack, so the pack keeps growing
  as people play.
- A `ps4_shaders.pack` placed in `/data/UnleashedRecomp/` overrides the one in the PKG, so new packs can be tried
  without reinstalling.

## Files and folders on the console

Everything lives in `/data/UnleashedRecomp/`:

| Path | What it is |
| --- | --- |
| `Install/` | Drop the game, update and DLC here for the first install |
| `game/`, `update/`, `dlc/`, `patched/` | The installed game (created by the app) |
| `Mods/` | Your mods, plus `ModsDB.ini` written by the mod menu |
| `Logs/boot.log`, `Logs/boot-previous.log` | Logs of this run and the last ones (attach these to bug reports!) |
| `config.toml`, `ps4_menu.ini` | Settings |
| `ps4_record/` | Shaders recorded for the next pack |
| `ps4_shaders.pack` | Optional override for the PKG's shader pack |
| `ps4_hung_pipelines.txt` | Pipelines skipped because they hung the GPU (delete to retry them) |

### Debug flags
Create an empty file with one of these names in `/data/UnleashedRecomp/` to change behaviour. They're for
troubleshooting; you shouldn't need them for normal play.

| File | Effect |
| --- | --- |
| `ps4_no_shadows` | Skip the shadow pass (same as the menu option) |
| `ps4_no_depth_only_draws` | Skip depth-only draws |
| `ps4_dma_clears` | Clear render targets with DMA instead of draws |
| `ps4_no_cube_maps` | Sample cube maps as blank |
| `ps4_no_prim_restart` | Turn off hardware primitive restart |
| `ps4_no_thread_priorities` | Leave thread priorities alone |
| `ps4_old_waits` | Use the C++ library's original thread waits (much slower) |
| `ps4_framegen_no_velocity` | Frame generation uses camera motion only |
| `ps4_profile` | Sampling profiler; writes the hottest functions to the log every 15 s (slows the game) |

## Troubleshooting

- **"No game found" / "No title update found":** check the files are directly inside
  `/data/UnleashedRecomp/Install/`, not in a sub-folder of a sub-folder.
- **"The title update doesn't match the game":** the update must be for the same region/version as your game.
- **Objects or characters invisible:** their shaders aren't in the pack yet. Play through the area, then send
  in the `ps4_record` folder.
- **Freezes / crashes:** open an issue with `Logs/boot.log` and `Logs/boot-previous.log` attached and say
  where it happened.
- **Frame generation looks jittery when turning fast:** that's the limit of generated frames; try
  *Interpolate* mode, or *Camera only* motion.

## Building from source

> Windows host with WSL is what the scripts are written for.

You need:
- [OpenOrbis PS4 Toolchain](https://github.com/OpenOrbis/OpenOrbis-PS4-Toolchain)
- The Unleashed Recompiled source with the PS4 backend, checked out next to this repository as
  `../UnleashedRecomp`, and your own game files for the recompiler (see the upstream README)
- WSL with Mesa built from source (patched RADV, for the shader pack; see `tools/shaders/build-mesa.md`)

```powershell
# Recompiled game code (slow, only needed once or when the recompiler output changes)
powershell -File tools\compile-ppc.ps1

# The app itself (incremental)
powershell -File tools\compile-app.ps1

# Package it (bump $Version in build.ps1 first)
powershell -File build.ps1 -Game
```

The output is `IV0000-UNLR00001_00-UNLEASHEDRECOMP0.pkg`.

### Repository layout

| Path | What it is |
| --- | --- |
| `build.ps1` | Links the ELF, writes `param.sfo`, and builds the PKG |
| `tools/compile-ppc.ps1`, `tools/compile-app.ps1` | Compile the recompiled game code and the app |
| `tools/shaders/` | The shader toolchain: Mesa/RADV patches, the pipeline packer and helpers |
| `src/` | The PS4 launcher / foundation code |
| `src-gputest/` | The small GPU test app that proved out raw GPU access on the console |
| `sce_sys/` | PKG metadata and icon |
| `PORTING.md`, `PS4_RENDERER.md` | Design notes for the port and the renderer |

## How it works

- **CPU:** the recompiled PowerPC game code compiles to x86-64 with OpenOrbis clang. Guest memory is mapped in
  pieces as the game allocates it, since the PS4 can't spare 4 GB up front.
- **OS layer:** PS4 versions of the game's OS modules (paths, logging, threads, timers, input, audio).
  The game's timer and thread-wait code were replaced with PS4-specific versions because the stock ones ran
  several times too slow on the PS4's kernel.
- **Graphics:** the game draws through *plume*, a small rendering layer with Vulkan/D3D12 backends. This port adds
  a PS4 backend that writes the GPU's command packets by hand, sets up textures and render targets in the GPU's
  native tiled layouts, and submits straight to the hardware.
- **Shaders:** the game's shaders become SPIR-V as on PC. On a PC, Mesa's RADV driver, pretending to be a GPU of
  the PS4's generation (GCN gfx7), compiles them, and a patch dumps the machine code and register setup into the
  shader pack. The console looks each pipeline up in the pack at runtime.

More detail in [PORTING.md](PORTING.md) and [PS4_RENDERER.md](PS4_RENDERER.md).

## Credits

- [hedge-dev](https://github.com/hedge-dev) and all contributors to
  [Unleashed Recompiled](https://github.com/hedge-dev/UnleashedRecomp), [XenonRecomp](https://github.com/hedge-dev/XenonRecomp)
  and [XenosRecomp](https://github.com/hedge-dev/XenosRecomp). This port is built entirely on their work.
- The [OpenOrbis](https://github.com/OpenOrbis) team for the PS4 toolchain.
- The [Mesa](https://www.mesa3d.org/) / RADV developers, whose compiler builds the PS4 shaders.
- Everyone who tested builds and sent logs and shader recordings.

## Legal

This is a fan project and is not affiliated with or endorsed by SEGA, Sony Interactive Entertainment or
Microsoft. *Sonic Unleashed* and all related trademarks belong to SEGA. No game assets or copyrighted game code
are distributed here; you must own the game to use this.

Unleashed Recompiled is licensed under the GPL-3.0; this port follows the same license. See the upstream
project's `COPYING` file.
