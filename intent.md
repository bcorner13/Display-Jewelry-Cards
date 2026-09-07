# Intent

## Goal

A countertop tiered display rack for 60×90mm earring cards, standing upright long-ways on
their short (60mm) edge. Rows are **actual discrete stepped tiers** — a real staircase, each
row its own physical step (Bradley's redesign, 2026-09-07) — not cuts into one continuous
ramp. Built and working as of 2026-09-07: **3 columns × 7 rows** (21 card slots, one tile).

Built as **two identical tiles** (same part printed twice) rather than one wide plate, for
transport. **Interlock: a separate connector strip** (`ConnectorStrip.FCStd`) — a
capsule-shaped bridge with a pin at each end, seating in matching pockets on each tile's
bottom — **2 connector pockets per side** (4 total, front+back pair on each edge, for
torsional rigidity — confirmed 2026-09-07), so any tile, leftmost/middle/rightmost, is the
same part. This supersedes the earlier snap-tab-built-into-the-tile idea, which Bradley
determined wouldn't hold well enough and removed (2026-09-06). Both the connector's
geometry and the tile's receiving pockets are done and audit-clean, built to sit flush with
the tile's bottom surface (a real tangent-vs-overlap gap bug was found and fixed here —
see `CLAUDE.md` hard rule 11).

## Constraints

* Must follow `CAD_STANDARDS.md` (mm units, manifold geometry, centered on the print bed,
  $36–$45 price target, 2.0–3.0mm default wall thickness, Silk-filament-friendly aesthetic).
* Printable without supports.
* Countertop use — freestanding, not wall-mounted.
* Card: 60mm wide (short edge, rests in the slot) × 90mm tall (standing height, "long ways"
  upright). **Fixed — not a design variable.**
* Column gap, row/column counts, tier geometry, tile width, and margins are the
  **adjustable levers** — make the layout work around the fixed 60×90mm card.
* `SlotWidth`=60.2mm (60mm card + 0.2mm clearance) — set.
* `Width` (tile footprint width) is now **derived** from `NumColumns*(SlotWidth+
  2*SlotSpacing)` = 198.6mm, not an independent value — keeps the footprint and the
  ramp/pocket extrusions from drifting apart (this had happened: a 1.4mm silent mismatch,
  fixed 2026-09-07).
* Connector: 40mm-long capsule, 5mm-radius pins 30mm apart, sits flush with the tile
  bottom via a stepped (deep round + shallow rectangular) pocket. `ConnectorClearance`=
  0.2mm/side — a first-pass estimate, tune after a test print/fit-check.

## Open questions (need Bradley's input)

* **Row pitch (`RowPitch`=35mm) and the row `Mode=Extent` setup** — working and verified by
  volume math, but the exact spacing values were chosen to fit the geometry rather than
  from an explicit ergonomic/aesthetic target. Worth a visual check once printed.
* **Wall/end margins** — how much material stays outside the outermost card column on each
  tile edge (currently whatever falls out of the `NumColumns`/`SlotWidth`/`SlotSpacing`
  math, not independently set).
* **Connector fit** — `ConnectorClearance`=0.2mm is unverified against an actual print;
  standard first-pass estimate per the project's material-dependent tuning convention (PLA
  vs. ASA shrink rates differ).
* **`Assembly.FCStd`** — container created but empty; still needs the two tiles + connector
  actually placed and joint-constrained to verify physical fit before calling the interlock
  design final.
* Any branding/engraving on the base, per the Silk-filament aesthetic guidance in
  `CAD_STANDARDS.md`?
