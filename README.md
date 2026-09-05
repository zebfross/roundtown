# Round Town

A no-fail, no-score toddler driving game. Drive a truck around an endless town,
honk, flip on the headlights (the whole town turns to night), run the siren,
and bump into things that react.

Single self-contained `index.html` — no build step, no dependencies.

## Controls (Xbox controller)

| Input | Does |
| --- | --- |
| Left stick / D-pad | Point it where you want to go. It turns and drives that way. |
| **A** | Honk |
| **X** | Change vehicle |
| **Y** | Headlights — toggles the town between day and night |
| **LB** / **RB** | Siren (flashing lights + sound) |
| RT | Go a bit faster |
| LT | Reverse |

Keyboard (for testing on the Mac): arrows/WASD to drive, Space honk, **V** change
vehicle (or **1**–**5** to pick one directly), **L** lights, **K** siren.

## Vehicles

**X** cycles through five. The name flashes up on screen when it changes, and the
choice is remembered in `localStorage`, so it starts up in whatever your kid
last picked.

| Vehicle | Top speed | Siren lights | Notes |
| --- | --- | --- | --- |
| Truck | 249 | amber | red cab, white box trailer |
| Car | 264 | amber | small and nippy |
| Fire truck | 237 | red / white | ladder on the roof, deepest siren |
| Police car | 274 | blue / red | fastest, fastest-flashing siren |
| Ambulance | 249 | red / white | red cross on the roof, two-tone siren |

Each has its own siren voice — waveform, pitch and warble rate are set per
vehicle in the `VEHICLES` table, so the fire truck growls and the police car
yelps. Changing vehicle while the siren is running re-voices it mid-wail.

Adding another is one entry in `VEHICLES`: give it a `draw(g)` that renders the
body facing **+X**, plus `len`/`wid` (shadow), `r` (how wide a berth it gives
when bumping things), `hx`/`hy` (headlights), `barX` (light bar) and `speed`.

## Running it on the Xbox

1. On the Mac, from this folder:
   ```
   python3 -m http.server 8733
   ```
2. On the Xbox, open the **Microsoft Edge** app.
3. Go to: **http://172.20.20.20:8733/**
4. Press any button to start. Set it as the homepage or bookmark it so it is
   two clicks next time.

The Mac has to be awake and on the same wifi. For something permanent, drop
`index.html` on any static host (Netlify/Cloudflare Pages/GitHub Pages) and use
that URL instead.

## The world

The town is an **unbounded grid of cells**, `BLOCK` units square, with roads
running along every cell boundary. There is no edge and no wrap — you can drive
in any direction forever.

Each cell's contents are derived purely from a hash of its `(i, j)` coordinates,
so a given location always generates identically, but the world never repeats.
Cells are built on demand around the truck and dropped once they are more than
`KEEP` cells away, so memory stays flat no matter how far you drive.

Cell kinds and roughly how often they come up: houses (~64%), field of cows
(~12%), park full of trees (~8%), barn with chickens (~8%), pond with ducks
(~8%).

## Design rules

Deliberate constraints, so it stays a toy and not a task:

- No fail states, no timers, no score, no lives, no menus.
- Every button does something pleasant. Nothing punishes a mash.
- **Nothing blocks the truck. Ever.** Not houses, not the barn, not trees,
  animals, bins, cones or balls. A 3-year-old cannot work out why the truck
  has stopped, so the truck never stops. Everything you hit reacts — it
  wobbles, makes a noise, throws up a puff of colour — and you keep going.
  The truck is drawn on top of buildings so it never disappears under a roof.
- Ponds slow you a little and splash, but you drive straight across.
- HUD sits inside a 5.5% inset so TV overscan does not clip it.

## Tweaking

Everything worth changing is at the top of the `<script>`:

- `VIEW_H` — how zoomed in the camera is. Lower = bigger truck. Currently 500.
- `MAXV` / `ACCEL` — speed and responsiveness.
- `BLOCK` / `ROAD_W` — size of a town block and width of the roads.
- `KEEP` — how many cells either side of the truck stay loaded before eviction.
- `PROP` — the bumpable things: `kick` is how far one gets knocked, `drag` is how
  much it slows the truck on impact (trees are 0.96, i.e. almost nothing).
- `genCell()` — the cell-kind probabilities and what goes in each kind.
- `SFX` — every sound is synthesised in Web Audio, no audio files.
