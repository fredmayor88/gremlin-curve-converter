# Curve Converter → Joystick Gremlin

**→ https://fredmayor88.github.io/gremlin-curve-converter/**

A one-file static tool. Design a response curve in `0-100` on both axes, read out the
`-1 … 1` numbers to type into Joystick Gremlin.

Conversion is the same on both axes:

```
gremlin = value / 50 - 1
value   = (gremlin + 1) * 50
```

Because both axes scale by the same factor, **a slope is the same number in either space** —
0.575 in `0-100` is 0.575 in Gremlin.

## Tabs

| Tab | What it does |
| --- | --- |
| **XY** | Add any number of points, type or drag them, get the Gremlin coordinates. |
| **Twitchy throttle (ACR)** | The actual job: taming the throttle in Assetto Corsa Rally. `x1 = 3`, `x2 = 90` pinned; you adjust `y1` and the slope. |
| **Twitchy throttle — free X** | Same fix with `x1` and `x2` unpinned, for when the pinned version isn't enough. |

On every tab, hovering the graph reads out the slope under the cursor, how much
faster/slower that is than 1:1, and the point itself in both coordinate spaces.

### Links carry the curve

The URL holds the whole setting, not just the tab, so a link reproduces exactly what you
were looking at. **Copy link** in the output header gives you the current one.

| Tab | Link |
| --- | --- |
| XY | [`#xy?p=3,29;90,79`](https://fredmayor88.github.io/gremlin-curve-converter/#xy?p=3,29;90,79) |
| Twitchy throttle | [`#twitchy?y1=29&slope=0.575`](https://fredmayor88.github.io/gremlin-curve-converter/#twitchy?y1=29&slope=0.575) |
| Free X | [`#twitchy-free?x1=3&y1=29&x2=90&slope=0.575`](https://fredmayor88.github.io/gremlin-curve-converter/#twitchy-free?x1=3&y1=29&x2=90&slope=0.575) |

Values from a URL go through the same snapping and clamping as typed input, so a malformed
or out-of-range link falls back to something legal instead of drawing a broken curve.

### What the two knobs do

**`y1` — dead range at the start of the pedal stroke**

- up → **less** dead range, the throttle bites sooner
- down → **more** dead range, more travel before anything happens
- resolution 0.5 in the `0-100` space

**`slope` — how fast the curve rises after the bite point**

- higher → rises **faster**, throttle comes in harder
- lower → rises **slower**, gentler and easier to modulate (the twitch fix)
- resolution 0.001; the `−`/`+` buttons nudge by 0.01

Resolution is enforced everywhere — dragging, typing and arrow keys all snap to it.

The tab also reports the curve **against a 1:1 linear response**: the default `0.575`
reads as *57.5% of 1:1 — 42.5% slower* over the main stretch, with the steep lead-in and
the top-end segment listed separately (they aren't 57.5%, and pretending otherwise would
be misleading).

Default preset is `y1 = 29`, `slope = 0.575`, which lands on:

```
p1 = (-0.94, -0.42)   gremlin  ->  (3, 29)        0-100
p2 = (0.8,   0.5805)  gremlin  ->  (90, 79.025)   0-100
```

Gremlin owns the endpoints `(-1, -1)` and `(1, 1)`; they're drawn hollow and stay out of the
output. Segments are linear.

## The experiment (ACR tab only)

The fix is three moves:

1. **Chop the dead part.** `y1` jumps straight over it — reaching 29% output used to cost
   29% of the pedal, now it costs 3%. That's **+26% of stroke** handed back.
2. **Slow it down.** Everything the game does between 29 and 79 output happens at
   **×0.575** — 1.74× more pedal for the same move.
3. **Sacrifice the top end.** Above 79 output it steepens to **×2.1**. Fair price: you're
   already committed up there.

Step 2 needs no model at all. Any stretch of throttle output costs exactly `1/slope` more
pedal travel, whatever ACR's curve turns out to be, because only your own curve is involved.
You're turning the game's volume down without knowing what it's playing. The one bet is
that its worst behaviour lives between p1 and p2 — outside that window your curve amplifies
instead of taming, which is precisely what steps 1 and 3 are spending.

The rest of this section prices that up against a guess at the curve. The signal chain is:

```
pedal travel → your Gremlin curve → the game's own curve → throttle
```

ACR's stage is **inferred, not measured**: throttle output in, how hard the car pushes out.
Deliberately simple — one dead zone, one ramp, straight to `(100,100)`:

```
push(u) = 0                      u ≤ dead
          ((u−dead)/(1−dead))^p  u > dead
```

`dead` isn't guessed. The correction implies it: `y1 − slope·x1` = **27.3%** output is where
the car must start responding for the fix to make sense.

`surge` marks where power starts building fast — about 47% output. **Not a traction limit
and not wheelspin**: control it smoothly and nothing breaks loose. It's simply where the
pedal matters most, so it's where you want travel.

All three parameters are adjustable and affect only the model, never your curve.

Two lines share the axes: pedal straight into the game, and pedal through your curve first.
Four tiles report what you'd notice from the seat, before → after:

| | Straight in | Through your curve | |
| --- | --- | --- | --- |
| Run-up to the surge | 20% | **31.6%** | 1.6× |
| Dead travel (under 5% push) | 30.7% | **5.9%** | 0.19× |
| Stroke covering 10-80% push | 51.2% | **80.8%** | 1.6× |
| Steepest run inside that band | 13.7% | 28.7% per 10% pedal | 2.1× |

That last row is the honest cost of the simple model: with the ramp running all the way to
`(100,100)`, the steep top segment of the correction lands on a still-live curve, so it
shows up as a spike. If ACR actually flattens off near the top — you're at full push before
full output — that spike isn't real. The model can't tell you which; driving it can.

## Running it

Open `index.html`. That's it — no build, no dependencies.
