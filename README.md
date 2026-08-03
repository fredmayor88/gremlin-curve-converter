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
| **Raw** | Add any number of points, type or drag them, get the Gremlin coordinates. |
| **Two-point (fixed X)** | `x1 = 3`, `x2 = 90` pinned. Adjust `y1` and the slope. |
| **Two-point (free X)** | Same, but `x1` and `x2` move too. |

Y steps by 1, slope steps by 0.01 — with the `−`/`+` buttons, by typing, or by dragging the
points on the graph. Default preset is `y1 = 29`, `slope = 0.575`, which lands on:

```
p1 = (-0.94, -0.42)   gremlin  ->  (3, 29)        0-100
p2 = (0.8,   0.5805)  gremlin  ->  (90, 79.025)   0-100
```

Gremlin owns the endpoints `(-1, -1)` and `(1, 1)`; they're drawn hollow and stay out of the
output. Segments are linear.

## Running it

Open `index.html`. That's it — no build, no dependencies.
