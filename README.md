# FF7 Rebirth Cutscene Editor

A desktop editor for Final Fantasy VII Rebirth's cutscenes. It reads the game's
Sequencer data and lets you cut shots, move and silence sound and music cues,
repoint music, and switch subtitles off — then builds an installable mod.

It does this without an Unreal editor and without re-cooking anything. Edits are
applied to the shipped assets byte for byte, which is why a scene that plays
correctly keeps playing correctly.

---

## What it can do

- **Remove shots** while keeping the lighting and streaming cues later shots
  depend on. Dropping a shot naively breaks the ones after it; the editor
  carries the cues forward. See `docs/FADES_AND_STREAMING.md` and
  `docs/LIGHTING.md`.
- **Move sound and music cues** by dragging, clamped to where the format allows
  them to go (`docs/CUE_DRAGGING.md`, `docs/CUE_RELOCATION.md`).
- **Repoint music** at a different track (`docs/MUSIC_REPOINTING.md`).
- **Switch subtitles off** using the game's own `Enable` flag, rather than
  blanking the text — which would leave a broken label on screen
  (`docs/SUBTITLES.md`).
- **Trim dialogue** to the seconds it should keep (`docs/TRIMMING_DIALOGUE.md`).
- **Preview the edit**, audio included. The preview mixes the real voice, SE and
  music sources with your edit applied, so what you hear is what the build
  produces (`docs/AUDIO_FOLLOWS_THE_CUT.md`).
- **Build and install** a `_P.utoc`/`_P.ucas` pak straight into your Paks
  folder (`docs/BUILDING_MODS.md`).

## What it cannot do

- Add new shots, animation or audio. It edits what is there.
- Re-time animation curves.
- Anything outside cutscenes.

---

## Requirements

| | |
|---|---|
| Windows | the game, the pak tools and Oodle are all Windows-only |
| Python 3.10–3.14 | from [python.org](https://www.python.org/downloads/), with "Add python.exe to PATH" ticked |
| FINAL FANTASY VII REBIRTH | installed, if you want setup to extract the assets for you |
| a `.usmap` for your game build | **required.** Not included here — it is the game's own type data. See [Game type mappings](#game-type-mappings-usmap) |
| ffmpeg | **optional** — setup offers to install one for you. Only preview audio and the waveform need it |
| UnrealReZen | **optional**, see [Building mods](#building-mods) below |
| Oodle | only to read the game's containers — see [Setting up from the game](#setting-up-from-the-game). Not needed if you set up from an existing export |

Python packages are installed for you on first run — PySide6 for the window,
numpy for audio mixing, and four small libraries for reading assets. They are
listed in `requirements.txt` if you would rather manage your own environment.

You do **not** need a C compiler, Node, FModel, or an Unreal install.

---

## Setup

Double-click **`CutsceneEditor.bat`**.

The first run sets things up and then opens the editor. It will offer your
Rebirth install if it can find one through Steam, and otherwise ask for a path.
You can point it at either:

- **your FINAL FANTASY VII REBIRTH folder** — it reads the game's own
  containers and extracts what it needs, or
- **an FModel or retoc export** — the `End` folder from a dump you already have.

The two routes differ in what they need and what they cost; see below.

From the command line, if you prefer:

```
py -3 firstrun.py "<game>"
py -3 firstrun.py --export "D:\dumps\End"
py -3 firstrun.py --game <path> --yes             # no prompts
py -3 firstrun.py --game <path> --extract-to E:\ff7   # assets on another drive
py -3 firstrun.py --limit 20                     # a quick smoke test
```

Setup also handles three optional tools — **ffmpeg** (preview audio and the
waveform), **UnrealReZen** (Build & install) and **AudioMog** (music in the
preview mix). Each is skippable and each only turns off the one feature it
gates.

If ffmpeg is missing, setup offers to install a self-contained build
(`imageio-ffmpeg`, about 80 MB) and puts it where the editor will find it. The
other two are binaries it cannot fetch, so for those it asks for a path — a
file or the folder holding it, now or later:

```
py -3 firstrun.py --ffmpeg "C:\tools\ffmpeg\bin\ffmpeg.exe" --rezen "C:\tools"
```

Setup writes `config.json`. That file is the only place your paths live — if a
drive letter changes, delete it and run setup again.

### Game type mappings (.usmap)

Setup asks for this first, and nothing works without it.

Every asset in Rebirth is stored in a form that cannot be read without a
description of the game's own types, and that description is a `.usmap` file.
**It is not included here** — it is the game's data, not this tool's, and not
ours to redistribute. It is not in the game folder either: it has to be dumped
from the game while it is running, with a mappings dumper. Anything producing a
UE4.26-era `.usmap` works; the ones people generally use are Dumper-7, the UE4SS
mappings dumper, and retoc's companion `jmap`.

Point setup at the file, or at the folder holding it:

```
py -3 firstrun.py --usmap "D:\ff7 modding\Mappings.usmap"
py -3 firstrun.py --usmap "D:\ff7 modding"
```

or leave it out and setup will ask. Either way it copies the file next to the
tool, so you are only asked once. Any filename is fine — dumpers are not
consistent about it — and setup checks the file really is an uncompressed
mappings dump before accepting it, rather than failing later inside a parse.

**Re-dump it when the game updates.** A `.usmap` from an older build will either
fail to parse an asset or, worse, parse it wrongly.

### Setting up from the game

Setup reads the game's own `.utoc`/`.ucas` containers and writes the loose assets
itself. Measured on a real install: **164,796 packages, about 10 GB, a few
minutes**, so `--extract-to` is worth using if your system drive is tight.

This needs **Oodle**, because that is what the containers are compressed with.
Oodle is proprietary and Rebirth **links it statically**, so — unlike most
Unreal games — there is no copy in the install to borrow. Two ways to get one:

1. **Let setup install `pyooz`**, an open-source decompressor. It offers this
   first and does it for you — one keystroke, a few hundred KB, nothing else
   needed.

   It is **GPLv3**, which is why it is offered rather than listed in
   `requirements.txt`: it goes into your Python, at your say-so, and is never
   shipped with this tool.

2. Or drop an `oo2core_<n>_win64.dll`, version 6 or newer, next to this README.
   It ships loose with a number of games: ELDEN RING (`oo2core_6`), DEATH
   STRANDING (`oo2core_7`), DOOM Eternal (`oo2core_8`), Indiana Jones and the
   Great Circle (`oo2core_9`). Searching your game folders for `oo2core` is the
   quickest way to find one, and Unreal Engine ships one too.

Setup prefers a DLL when it finds one and falls back to `pyooz`, so a 32-bit or
corrupt DLL is not a dead end.

### Setting up from an existing export

Already have an FModel or retoc dump? Point setup at its `End` folder instead.
None of the above applies — no Oodle, no extraction, no 10 GB.

### Then the timelines

Either way, setup finishes by extracting all **755 cutscene timelines at about a
second each, so roughly fifteen minutes**. Interrupting is safe; running setup
again continues where it stopped rather than starting over.

### Actors: pins, or sprites

The stage preview draws each actor as a **pin** — a vertical line with a head
dot. That is the default and usually the clearer read: pins never occlude one
another and never imply a facing the data does not have.

`extras/character-sprites/` holds the alternative, a sprite atlas of the party
rendered from the 1997 field models at 16 yaw steps. Copy `chars.png` and
`chars.json` up into this folder to use it, delete them to go back. It sits in
a subfolder precisely so it is inert until you choose it — `sprites.py` only
ever looks in the tool root. That folder's own README has the details and the
licensing note.

### Audio arrives as you go

Setup deliberately builds **no** audio. Rendering every scene's voice, music and
waveform peaks produces around 2.8 GB and takes hours. Instead the preview is
mixed from the per-cue sources the first time you open a scene and cached after
that, so the first play of a cutscene takes a moment and later ones are instant.

Caches live under `browser_v2/timelines/audio/`. Delete that folder any time to
reclaim the space; it rebuilds on demand.

### Disk

| | |
|---|---|
| the tool itself | ~6 MB |
| extracted assets (game setup only) | ~10 GB, 164,796 packages |
| 755 timelines + manifest | ~135 MB |
| audio and waveform caches | grows as you use it, up to ~2.8 GB |

Setup checks the target drive has room before it starts extracting, rather than
failing part-way through.

---

## Building mods

**UnrealReZen is not bundled.** It is licensed under the GPL v3, and shipping
the binary without also shipping its source would not honour that licence. So:

1. Get it from [rm-NoobInCoding/UnrealReZen](https://github.com/rm-NoobInCoding/UnrealReZen).
2. Put `UnrealReZen.exe` next to this README.
3. Restart the editor — **Build & install** lights up on its own.

Without it, everything except building works; the Build button explains itself
rather than failing.

Oodle is not redistributable either, but it cannot be copied out of the game —
Rebirth links it statically. See [Setting up from the game](#setting-up-from-the-game)
for the two ways to supply one, and note it is only needed to read the
containers, not to build a mod.

### Checking the extraction is correct

If you have both a game install and an FModel export of the same build, you can
prove the container reader agrees with FModel byte for byte:

```
py -3 gameextract.py verify "<game folder>" "<your export End folder>"
```

It samples packages evenly across every area, compares each against the same
file in your export, and prints a verdict. This has been run: **400 packages
across 10 areas, all 400 byte-identical, 0 differing** — and a 300-file sample of
a written-out area was identical on disk too.

One known difference between the two routes. Voice-line durations come from
`Properties.Duration` in FModel's JSON, which cannot be read out of the asset:
the parse dies on an unmapped property, and a fixed-offset byte read agreed with
FModel on only 93.8% of 2,500 assets, which is worse than not having one. So
when setting up from the game, the duration is derived from the embedded HCA
audio header instead. Checked against FModel on 2,500 assets: **all agree within
0.06s**, none further off, none bit-exact. That is one or two frames on a bar
that only shows how long a line runs, and it is the only thing in a built
timeline that differs between the two routes.

The subtitle reader has its own check, against FModel's own JSON exports:

```
py -3 txtres.py verify "<export>/Content/Text/US"
```

---

## If something goes wrong

**"No Python found"** — install Python 3.10 or newer and tick "Add python.exe to
PATH".

**The install step fails with "no matching distribution"** — that Python is
outside the 3.10–3.14 range PySide6 publishes wheels for. Installing Python 3.13
alongside it is the quickest fix; the launcher prefers whichever interpreter
already works.

**"Nothing was found under Cut/Game/"** — the containers mounted but do not hold
cutscenes. Check you pointed setup at Rebirth and not another Unreal game.

**No subtitle text in the editor** — the caption tables did not generate. Setup
says so when it happens. An FModel JSON export of `Content/Text/US` dropped into
the extracted tree fixes it; the tool never overwrites an export you provide.

**No preview audio, or no waveform** — ffmpeg is not on PATH. Either put it
there, or re-run setup and give it the path; the editor adds that folder to PATH
on startup.

**Paths are wrong after moving a drive** — delete `config.json` and run setup
again.

**"does not start with the .usmap magic"** — that file is not a mappings dump.
**"is compressed"** — re-export it without compression. **Assets parse but the
data looks wrong** — your `.usmap` is probably from a different game build;
re-dump it.

**A scene looks wrong after an edit** — `docs/LESSONS.md` is the accumulated list
of ways this has gone wrong before, and is the first place to look.

---

## How it is put together

| | |
|---|---|
| `CutsceneEditor.bat` | the launcher: finds Python, installs packages, runs setup if needed |
| `firstrun.py` | setup |
| `toolconfig.py` | where paths come from — flags, then `config.json`, then discovery |
| `gameextract.py` | reads the game's containers into a loose Zen export tree |
| `txtres.py` | reads `EndTextResource`, the subtitle tables |
| `batch_build.py` | extracts every cutscene timeline and the manifest |
| `extract_timeline.py` | one cutscene's Sequencer data to JSON |
| `patcher.py` | byte-level edits to the shipped assets |
| `build_mod.py` | stages the edits and packs them |
| `editor_qt/` | the editor — `core/` is the logic, `ui/` is the window |
| `vendor/UE4Parse` | an unmodified copy of pyUE4Parse; `vendor/UE4Parse-VERSION.txt` names the exact upstream commit |
| `extras/character-sprites/` | an opt-in sprite atlas, off by default — see below |
| `docs/` | the design notes, written as the thing was figured out |

Worth reading if you intend to change anything: `docs/LESSONS.md`,
`docs/EDITING.md`, `docs/CHAIN_FORMAT.md`, and `editor_qt/README.md`.

### Tests

```
py -3 -m pytest editor_qt/tests/test_core.py -q
```

Most of the suite tests real extracted data, so what you should see depends on
how much you have extracted:

| | |
|---|---|
| fresh unzip, no `.usmap` yet | ~99 passed, ~145 skipped |
| `.usmap` in place, nothing extracted | ~105 passed, ~139 skipped |
| a partial extraction | ~107 passed, ~137 skipped |
| all 755 scenes | ~242 passed, ~1 skipped |

**No errors and no failures in any of those.** A test whose scene is not in your
extraction skips and names the scene; errors would mean something is actually
wrong.

**Watch the skip count, not just the pass count.** A fixture that quietly goes
missing turns tests into skips rather than failures, which is exactly how it
hides.

The suite tests against real extracted data, and none ships: no `.usmap` and no
staged assets, because both are the game's. So a fresh copy skips a good deal
until you have run setup, and the tests that patch real assets stay skipped
unless you stage some yourself.

---

## Licence

**MIT** — see `LICENSE`. Use it, change it, build it into your own tool, ship it
in a mod of your own. The one condition is the usual MIT one: keep the copyright
line and the licence text with it, so the credit travels.

(In a release zip the MIT text is `LICENSE`. In the development tree it is
`MIT-LICENSE.txt`, because `LICENSE` there is UnrealReZen's GPL v3, left from
when its binary sat in that folder. `makerelease.py` swaps them on the way out.)

If you are lifting a piece rather than the whole thing, these are the parts most
likely to be worth taking on their own, and a line in your readme or mod page is
all the credit that is asked:

| | |
|---|---|
| `txtres.py` | reads `EndTextResource` — every subtitle in the game, keyed by label. Verified byte-identical to FModel across all 142 areas |
| `gameextract.py` | reads the game's own containers into loose Zen assets, including the pyUE4Parse fixes needed to make that work on Rebirth |
| `cutscenelist.py` | `CutSceneListLevel`, the table that chains one cutscene to the next |
| `patcher.py` | byte-level edits to shipped assets, cues and subtitles included |

### What the licence does not cover

| | | |
|---|---|---|
| [pyUE4Parse](https://github.com/MinshuG/pyUE4Parse) | MIT | bundled in `vendor/`, an unmodified pure-Python copy. `licenses/pyUE4Parse-MIT.txt` |
| [UnrealReZen](https://github.com/rm-NoobInCoding/UnrealReZen) | GPL v3 | **not bundled**; `licenses/UnrealReZen-GPL-3.0.txt` is its licence, not this tool's |
| Oodle | proprietary | **not bundled**, and not in the game either — Rebirth links it statically |
| [pyooz](https://pypi.org/project/pyooz/) | GPLv3 | **not bundled and not a dependency**; used only if you install it yourself |
| [ffmpeg](https://ffmpeg.org/) | LGPL/GPL | **not bundled**; expected on PATH |
| AudioMog | — | **not bundled**; optional, for BGM extraction |
| a `.usmap` | — | **not included.** The game's own type data; you supply one, see [Game type mappings](#game-type-mappings-usmap) |

Anything the tool reads out of, or writes into, FINAL FANTASY VII REBIRTH is
Square Enix's, and none of it is included here.

## Credits

Built by Aden.

pyUE4Parse by MinshuG. UnrealReZen by rm-NoobInCoding. The `EndTextResource` and
`CutSceneListLevel` formats were worked out here, with FModel's output as the
reference to check against.
