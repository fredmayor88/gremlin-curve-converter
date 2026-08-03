# Curve Converter → Joystick Gremlin

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

Below the editor there's a second graph modelling what the pedal actually does once the
game has had its turn. The signal chain is:

```
pedal travel → your Gremlin curve → the game's own curve → throttle
```

The game's curve is a stand-in, not a measurement:

```
f(x) = (e^(k·x³) − 1) / (e^k − 1)      k defaults to 3
```

Normalised through `(0,0)` and `(1,1)`. The `x³` keeps the first third nearly flat; the
exponential makes the top end vicious. `k` is adjustable — raise it until the grey line
matches how the car actually feels. It only affects the model, never your curve.

Two lines are drawn on the same axes: the pedal going straight into the game, and the pedal
going through your curve first. The table underneath gives the numbers at a few pedal
positions.

## Running it

Open `index.html`. That's it — no build, no dependencies.
