# Controls

Everything the editor responds to, as it currently stands.

---

## Getting around

| | |
|---|---|
| `Space` | play / pause |
| `Home` / `End` | jump to the start or the end of the scene |
| `←` / `→` | step one frame |
| `Shift` + `←` / `→` | step thirty frames |
| `F` | zoom to fit the whole scene |
| `+` / `−` | zoom in / out |
| `M` | mute |
| drag in the ruler | scrub the playhead — the strip along the top of the timeline |
| `Ctrl` + wheel | zoom, centred on the pointer |
| `Shift` + wheel | scroll sideways |

The toolbar carries the same transport: the scene picker, **▶ Play**, **⏮ ⏭**,
and **Fit − +**. The clock on the right is the playhead position.

---

## Selecting

**Click a block** to select it. The Details panel then describes it, and the
keyboard and right-click menu act on it.

**Click again to reach what is underneath.** Cues are instantaneous and drawn
as a quarter-second sliver, so two on the same frame cover each other exactly.
Repeated clicks step down through the pile and wrap back to the top, the way
overlapping objects behave in a drawing program. Without this a buried cue
could not be selected at all — which is how `EV_CORLE_0100` once shipped with a
flower lei still attached.

**`Shift` + drag** sweeps out a time range. Plain dragging is reserved for
moving a cue, so the range gesture needs the modifier.

**Clicking a block on a range row** selects that block's own span as the range.

`Esc` clears the selection.

---

## Moving a cue

**Drag it.** The move is proposed live and snaps to the frame grid; a short
drag stays a click, so you cannot nudge something by accident.

**`Alt` + drag** turns snapping off, for a cue that has to sit between two
places the grid would prefer.

**`R`** sends the selected cue back to the tick the scene authored it at.

Not every block can move. The timeline will not propose a move it cannot make.

---

## Editing

| | |
|---|---|
| `Delete` / `Backspace` | switch the **selected cue** off — press again to put it back |
| `Delete` with nothing selected | cuts the **selected range** instead |
| `Shift` + `Delete` | always cuts the range, whatever is selected |
| `Ctrl` + `M` | point the selected music trigger at another track |
| `Ctrl` + `Z` | undo |
| `Ctrl` + `Y` or `Ctrl` + `Shift` + `Z` | redo |

Two things `Delete` does that are worth knowing. An **attach** cue is not
removed but turned round — the build flips its own Detach flag, so the status
bar says "detached" rather than "deleted". And some cues are **protected**:
the game needs them to progress, so the editor refuses and suggests cutting
their time range instead.

### Right-click

The menu only offers what applies to the block under the pointer:

- **Point this trigger at another track…** — on a music trigger
- **Play *x* again (what the scene authored)** — if you have already repointed it
- **Switch this cue off** / **Put this cue back**
- **Send it back to where the scene put it** — if you have moved it

Right-click selects the block it is over and does *not* cycle through a pile.
Having clicked down to a buried cue, right-clicking has to mean that one.

---

## The three toolbar switches

**Split dialogue by speaker** (`Ctrl+D`) gives every character their own
dialogue row. Lines overlap constantly — a reaction lands under the tail of the
line it answers — and on a single row the buried block cannot be clicked, so it
cannot be dragged or switched off. A view preference; it changes nothing in the
plan and is remembered between sessions.

**Carry fades & streaming** (*on by default*) stops a cut where the scene's
fade to black begins rather than on a hard frame. The fade lands on the join,
so the level-streaming commands that preload the next scene still fire and you
never see the footage they sit in. The edit ends up longer than the cut by the
fade's length; the timeline draws that stretch in black.

**Restore preload lead** (*off by default*) moves a Preload the cuts never
touched, when a join lands inside the window the sky needs to load in. Unlike
the switch above it is a guess about engine bandwidth and has not been
confirmed in game, which is why it is off unless you ask for it.

---

## Actors

The **Actors** panel lists everyone the scene shows or hides.

Set an actor to **Normal**, **Show** or **Hide** to pin them for the whole
scene. To pin only part of it, drag a range on the timeline first, then use
**Show ▸** or **Hide ▸** on that actor's row. Each pin can be cleared on its
own, or dropped all at once.

---

## Preview

| | |
|---|---|
| `Enter` | render the current edit as a keepable `.ogg` |

Preview audio is mixed from the real voice, SE and music with your edit applied
— disabled cues left out, moved cues at their new tick, a repointed trigger
playing the track you chose. It is built per scene the first time you open one
and cached after that, so the first play takes a moment.

The mix stays on the **original** timeline and the playhead skips the cuts,
rather than the cuts being rendered into the file. One place skips; that is
deliberate.

Needs ffmpeg. Without it, preview audio and the waveform are the only things
missing.

---

## Panels and layout

Eight docks: **Timeline**, **Cutscenes**, **In frame**, **Top-down**,
**Lip sync**, **Details**, **Actors**, **Build log**. Every one can be moved,
floated, stacked or closed.

**View ▸ Panels** toggles any of them, including the toolbar. **Show every
panel** brings back anything closed without disturbing your arrangement;
**Reset window layout** puts everything back as it started. Your layout is
remembered between sessions.

---

## Saving and building

| | |
|---|---|
| `Ctrl` + `S` | save the edit plan to the `plans` folder |
| `Ctrl` + `B` | save the plan, patch the assets, pack and install the mod |

Build & install needs the extracted assets, a `.usmap`, the game's Paks folder
and UnrealReZen. Without them the button explains what is missing rather than
failing. Progress goes to the **Build log** panel.

---

## A note on where keys fire

The chorded shortcuts — `Ctrl+Z`, `Ctrl+S`, `Ctrl+B`, `Ctrl+M`, `Ctrl+D` — work
anywhere in the window.

The bare keys — `Space`, `Delete`, the arrows and the rest — deliberately do
not fire while you are typing in a search box. Otherwise a space in a query
would pause playback and `Delete` while correcting a typo would switch a cue
off.

---

## Not available from the editor

**Trimming a line** to a shorter length is supported by the plan format and by
`build_mod.py`, but nothing in the window triggers it yet. The timeline draws
the silenced tail of an already-trimmed line as a ghost, so trims made another
way do show up.
