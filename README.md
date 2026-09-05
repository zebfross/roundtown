# Round Town

**Play it: https://zebfross.github.io/roundtown/**

A no-fail, no-score toddler driving game. Drive around an endless town in one of
five vehicles, honk, flip on the headlights (the whole town turns to night), run
the siren, and bump into things that react.

One `index.html` — no build step, no dependencies. Two recorded sounds live in
`assets/`; without them the game falls back to its synthesised ones and still
works from the HTML file alone.

## Controls (Xbox controller)

| Input | Does |
| --- | --- |
| Left stick / D-pad **up** / **down** | Drive forwards / backwards |
| Left stick / D-pad **left** / **right** | Steer, and it turns on the spot when stopped |
| **A** | Honk |
| **X** | Change vehicle |
| **Y** | Headlights — toggles the town between day and night |
| **LB** / **RB** | Siren (flashing lights + sound) |
| Left / right, then **A** | Answer a letter-game question. **Y** repeats it, **B** drives on. |
| RT | Go a bit faster |
| LT | Reverse |

Keyboard, for working on it on the Mac: arrows/WASD to drive, Space honk,
**V** change vehicle (or **1**–**5** to pick one directly), **L** lights,
**K** siren.

The on-screen hints follow whatever is actually in your hands: keycaps on a PC,
Xbox glyphs on a controller, PlayStation shapes on a DualSense. Chrome only
reveals gamepads after a button press, so it starts on keyboard and flips the
instant a controller is touched -- on the Xbox that happens before the HUD is
ever seen. An idle connected pad does not steal the hints back from the
keyboard; only actual input switches them.

### Movement

Two systems, because which one suits a small child is not obvious:

- **`turn`** (default) — up and down are forward and backward thrust, left and
  right steer. Steering works at a standstill, so it can always be pointed at
  something without having to be moving first. Much easier for lining up on a
  duck and shoving it into a pond.
- **`point`** — the original. Push the stick the way you want to go and it turns
  to face there and drives. Fewer things to coordinate, but awkward for precise
  nudging.

Switch between them on the **settings screen** — press the **Menu** or **View**
button on a controller, or **Esc** on a keyboard. It pauses the world while it
is open, and nothing reaches the game underneath, so a stray press cannot honk
or change vehicle while you are in there. Close with **B** / **Menu** / **Esc**.

`?drive=point` or `?drive=turn` on the URL still works, and either way the
choice is saved until changed again.

Adding another setting is one entry in the `SETTINGS` array — give it a `label`,
a list of `options` with a `name` and `hint` each, and a `get`/`set` pair. It
renders and becomes navigable with no other changes. A sound on/off row would be
the obvious next one.

Reverse is 75% of forward thrust and capped at half top speed. `TURN` at the top
of the script is the steering rate in radians/sec.

## Running it on the Xbox

1. On the Xbox, open the **Microsoft Edge** app.
2. Go to **https://zebfross.github.io/roundtown/**
3. Press any button to start.

Typing that with a controller is miserable, so do it once and then set it as
Edge's homepage, or bookmark it. After that it is two clicks.

### Working on it locally

    python3 -m http.server 8733

Then `http://172.20.20.20:8733/` from the Xbox on the same wifi, or
`http://localhost:8733/` on the Mac. Worth it while changing things, since it
skips the deploy round-trip.

## Deploying

Hosted on GitHub Pages from `main`, so a push is a deploy:

    git push

About a minute to go live. No build and no CI — Pages serves the repo root
as-is.

## Vehicles

**X** cycles through five. The name flashes up on screen when it changes, and
the choice is saved in `localStorage`, so it starts up in whatever your kid last
picked — including on the start screen, which shows their vehicle.

| Vehicle | Top speed | Siren lights | Notes |
| --- | --- | --- | --- |
| Truck | 249 | amber | red cab, white box trailer |
| Car | 264 | amber | small and nippy |
| Fire truck | 237 | red / white | ladder on the roof, deepest siren |
| Police car | 274 | blue / red | fastest, and a recorded siren |
| Ambulance | 249 | red / white | red cross on the roof, two-tone siren |

Top speed is set by `ACCEL * speed / 1.7`, not by the `MAXV` clamp — `MAXV` is
only an upper bound and is never reached in normal driving.

Adding another is one entry in `VEHICLES`: give it a `draw(g)` that renders the
body facing **+X**, plus `len`/`wid` (shadow), `r` (how wide a berth it gives
when bumping things), `hx`/`hy` (headlights), `barX` (light bar) and `speed`.

## Sound files

`SAMPLE_SRC` at the top of the script maps a recording to each slot:

| Slot | File | Used for |
| --- | --- | --- |
| `horn` | `assets/car-horn.mp3` | the horn, on **every** vehicle |
| `police` | `assets/police-siren.mp3` | the police car's siren only |

Drop a different mp3 in at the same path to change a sound — no code edit
needed. If a file is missing or fails to decode, that sound quietly falls back
to the synthesised version in `SFX`.

`gain` in that table is the volume. Both current files are mastered to full
scale, so they are turned down there (horn 0.30, siren 0.22) rather than being
left to dominate the mix. Every other sound in the game is synthesised in Web
Audio, no files involved.

Anything marked `loop:true` goes through `seamlessLoop()` when it loads. A clip
that simply loops end-to-start clicks once per cycle unless its ends happen to
match; this trims the last 60 ms and crossfades them back over the start. On the
supplied siren that took the jump at the join from 1.50 down to 0.07.

To give the recorded horn to just one vehicle instead of all of them, add a
`hornSample:'horn'` key to that vehicle and have `doHorn()` check it, the same
way `sirenSample` works.

## Letter games

Purple **?** pads are scattered around the town, roughly one every five blocks.
Drive onto one and it asks a question. The world pauses while it is up.

It names the thing and asks for its first letter — *"What does **TREE** start
with?"* — with a big picture of it and three letters to choose from.

The other direction (showing a letter and asking which of three pictures starts
with it) is harder: it needs every picture named and its initial worked out, all
held in mind at once. It is off by default. Turn it on with the **QUESTIONS**
row in settings once that gets easy.

Everything asked about is drawn from artwork already in the world — cow, duck,
chicken, tree, ball, cone, bird, car, truck, ambulance, house, barn, pond — so a
child who cannot read yet still knows what they are looking at. Three choices,
always one right answer and two distractors with different first letters.

**There is no losing.** A wrong pick wobbles the card, makes a soft noise and
asks again. Nothing is scored, nothing is counted, and the pad can be driven
over as many times as they like — it re-arms once you have driven off it, and
picks a fresh question each time.

Questions are read aloud through the browser's speech synthesis, which matters a
lot before a child can read. **Each option is also spoken as it is highlighted**,
so the letter shapes do not have to be known yet — a child can scroll along and
match by sound. **Y** (or **R** on a keyboard) repeats the question. If speech is unavailable the game carries on silently; turn it off
deliberately with the **TALKING** row in settings.

Add more things to ask about by appending to `QUIZ_ITEMS`: a word, a scale `s`
to normalise its size, and an `art()` that draws it centred on the origin at
roughly 30 units. `propArt()`, `vehArt()`, `boxArt()` and `pondArt()` are there
to reuse the existing drawings.

## The world

The town is an **unbounded grid of cells**, `BLOCK` units square, with roads
running along every cell boundary. There is no edge and no wrap — you can drive
in any direction forever.

Each cell's contents come from a hash of its `(i, j)` coordinates, so a given
location always generates identically, but the world never repeats. Cells are
built on demand around the vehicle and dropped once they are more than `KEEP`
cells away, so memory stays flat no matter how far you drive.

Cell kinds and roughly how often they come up: houses (~64%), field of cows
(~12%), park full of trees (~8%), barn with chickens (~8%), pond with ducks
(~8%).

One consequence of eviction: knock a cow across a field, drive ten blocks away
and come back, and the cow will be where it started. Keeping that state would
mean unbounded memory. Raise `KEEP` to trade memory for persistence.

## Design rules

Deliberate constraints, so it stays a toy and not a task:

- No fail states, no timers, no score, no lives, no menus.
- Every button does something pleasant. Nothing punishes a mash.
- **Nothing blocks the vehicle. Ever.** Not houses, not the barn, not trees,
  animals, bins, cones or balls. A 3-year-old cannot work out why the truck has
  stopped, so it never stops. Everything you hit reacts — it wobbles, makes a
  noise, throws up a puff of colour — and you keep going. The vehicle draws on
  top of buildings so it never disappears under a roof.
- Ponds slow you a little and splash, but you drive straight across.
- HUD sits inside a 5.5% inset so TV overscan does not clip it.

## Tweaking

Everything worth changing is at the top of the `<script>`:

- `VIEW_H` — how zoomed in the camera is. Lower = bigger vehicle. Currently 500.
- `ACCEL` — responsiveness, and the thing that actually sets top speed.
- `BLOCK` / `ROAD_W` — size of a town block and width of the roads.
- `KEEP` — how many cells either side stay loaded before eviction.
- `SAMPLE_SRC` — the recorded sounds and their volumes.
- `VEHICLES` — the five vehicles: looks, speed, siren colours and siren voice.
- `PROP` — the bumpable things: `kick` is how far one gets knocked, `drag` is how
  much it slows you on impact (trees are 0.96, i.e. almost nothing). Buildings
  have no drag at all — they only wobble.
- `genCell()` — the cell-kind probabilities and what goes in each kind.
- `SFX` — the synthesised sounds.
