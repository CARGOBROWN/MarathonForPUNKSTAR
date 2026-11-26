# MarathonForPUNKSTAR

A fork of [VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX), itself a fork of
[VIctory](https://github.com/Marvisak/VIctory): a compatibility layer that runs an Xbox Series title
natively on Windows from files the user supplies. This one is about **playing on an AMD Radeon
RX 7900 XTX**: speed first, without crashes, then the picture.

**This repository contains no game files, code or assets.** You need your own legally obtained copy
of the game. It is not affiliated with or endorsed by the game's developer or publisher.

## Where it stands (2026-11-26)

Tested on one machine only: Windows 11, AMD Ryzen 7 5800X3D, 64 GB RAM, Radeon RX 7900 XTX 24 GB,
diagnostics build, second launch of the build.

| | |
|---|---|
| Menus, videos, loading screens | 30 fps |
| Prison cell, standing still | 65-73 ms a frame (14-15 fps) |
| Open world, played | median 89 ms a frame (11 fps); one frame in ten over 135 ms; stutters remain |
| Longest session played | 32 minutes, no device loss |
| Memory | about 40-42 GB of private memory, steady after 10 minutes: **64 GB of RAM needed** |

The window's caption tells where a frame's time goes: peak frame, CPU and copies, copying back,
the card's busy share, video memory, RAM, free commit, and pipelines and textures pending
(`VICTORY_TITLE_STATS=0`: the frame rate only).

What is known to be wrong:

- **Memory.** Free commit can still fall to about 2 GB with a page file sized by Windows.
- **Trees flicker** in the open world, appearing and disappearing. Not memory alone: it stays
  with textures let go only below 2 GB of free commit, after a minute unused. Being looked at.
- **Stutters.** About one second in five has a frame over 150 ms; 10 fps can feel like 3-4.
  Being looked at.
- **Streaming.** The title asks for files in bursts of thousands; requests wait up to a second or
  more, and models (people, trees) can stay at their lowest detail for a while. Being looked at.
- The open world has dips of 150-450 ms a frame while it streams.
- Hair can be drawn over faces in some shots.
- Rooms with little direct light are dark unless the title's ray tracing is turned on (`-RayTracing`).
- The first run of a new build or driver is slow for some minutes while pipelines are made.

## What is different here

On top of VIctory-7900XTX (what that fork does, and why, is in [docs/findings.md](docs/findings.md)):

- **Raw accesses to buffers picked from the view heap stay inside the view found there**
  (`VICTORY_PICK_BOUND`, on). On AMD a raw access through a structured or typed view is held to the
  element index alone, and a byte offset read from memory that holds something else reached up to
  4 GiB past the view; Radeon GPU Detective named the load. On this machine, without the bound every
  run that moved through the world lost the device within 4 to 10 minutes; with it none did.
- **A copy that meets one of the title's guard pages puts the guard back** before it fails, as the
  kernel's copy leaves it.
- **Memory under pressure:** host copies of textures, and empty blocks of the card's memory, go
  when the PC has little left to commit (`VICTORY_TEXTURE_COMMIT_MB`).
- **Pull requests of VIctory-7900XTX taken early:** #24 (UwUCammie), #68 (zocram4cc), #77
  (Adika713; `scripts\play.ps1` sets `VICTORY_WINDOW_BUDGET_MB=2048`), #87, #92 (TheHoracio:
  windows of title memory in the card's memory), #98 (Joly0: streaming stores for big copies),
  #105 and #108 (SousaTrashBin: card steps without the waits behind placed copies; a loop limit
  for the ray march `39f0e9aeb156a61c`, `VICTORY_SHADER_LOOP_LIMITS`) and #96 (kalijerry:
  dynamic descriptors from eight sharded pools). Upstream's main is merged up to 85d5641.
- **The caption's performance figures**, and stopwatches of the texture side of the pass over the
  view heap (diagnostics).

## How to run it

You need Windows 10/11 x64, a CPU with AVX2, a Direct3D 12 GPU with shader model 6.6 and mesh
shaders, Rust (the toolchain is pinned), the MSVC build tools, and your own dump of the title.

```powershell
git clone https://github.com/CARGOBROWN/MarathonForPUNKSTAR
cd MarathonForPUNKSTAR
copy victory.example.toml victory.toml      # then set your paths, outside this folder
cargo build --locked --profile diag -p victory --features diagnostics
target\diag\victory.exe shaders             # every shader translated ahead, a few minutes
scripts\play.ps1 -Exe <path of your copy's executable>
```

`scripts\play.ps1 -Safe` leaves out the newest, faster ways; try it first when something is wrong.
`-RayTracing` turns the title's ray-traced indirect light on, about a third slower. Stay windowed.

Commands, debugging switches and upstream's own description: [docs/README-fxceless.md](docs/README-fxceless.md).
Every `VICTORY_...` switch is described in a comment where it is read.

## Reporting a test

Open an issue with text only: GPU and its memory, driver, CPU, RAM, page file size, the commit
(`git rev-parse --short HEAD`), how long it ran and how far you got, the caption's figures in a menu,
indoors and in the open world, and the last lines of the log (check that they hold no personal path
or name). **Never attach** game files, shaders, saves, screenshots, videos or captures.

## Credits

This is a tree of other people's work with a little on top. The history is kept, and commits taken
from elsewhere name the commit and its author.

- **[VIctory-7900XTX](https://github.com/avatarmanz/VIctory-7900XTX)** by avatarmanz: the base of
  this fork, and most of its speed on RDNA 3.
- **[VIctory](https://github.com/Marvisak/VIctory)** by Marvisak and its contributors: the loader,
  the runtime services, the Direct3D 12 back end and the shader recompiler.
- **[fxceless/victory-fork](https://github.com/fxceless/victory-fork)**: the fork VIctory-7900XTX
  started from, and the people whose pull requests it carries.
- **[DexNom/VIctory](https://github.com/DexNom/VIctory)**,
  **[Belobug/VIctory-Belobug-AMD](https://github.com/Belobug/VIctory-Belobug-AMD)** (with
  romainhedouin), **[FQY55kai/ZJOU-VICTORY](https://github.com/FQY55kai/ZJOU-VICTORY)**,
  **[myokex/victory-amd](https://github.com/myokex/victory-amd)** (with Spexivus, joroyt, tightfall
  and Joshua Odom), **Geckomacabre** and **cxy0072019-sketch**: as credited by VIctory-7900XTX.
- **TG0000**: the guard page that stays one (VIctory 79c9b93).
- **[KLBF Lab](https://github.com/PastiiJay/klbf-lab)** by PastiiJay: where this fork's own work
  began; that earlier work is kept in this repository's other branches.
- **[XCL](https://github.com/ivall/xcl)** (GPL-2.0-only) was read for how the same title behaves
  under another layer; no code of it is here.
- Changes of this fork and of VIctory-7900XTX were made with [Claude Code](https://claude.com/claude-code).

## License

GPL-3.0-or-later, as VIctory. See [LICENSE](LICENSE); the one file that is not covered is named in
[docs/asset-policy.md](docs/asset-policy.md).
