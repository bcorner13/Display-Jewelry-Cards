# Intent

## Goal

A countertop tiered display rack for 60×90mm earring cards, standing upright long-ways on
their short (60mm) edge. Rows are **tiered/cascading** — each row sits higher and further
back than the one in front (like theater seating), so every row's cards stay visible from
the front. Built and working as of 2026-09-06: one continuous sloped shelf, patterned into
**3 columns × 4 rows** (12 card slots, one tile).

Built as **two identical tiles** (~200mm each — same part twice) rather than one wide plate,
so each printed piece stays small enough to transport. **Interlock mechanism is open again**
— a snap-tab friction peg/pocket was built and verified geometrically, but Bradley
determined **it won't actually hold the two tiles together** and removed it (2026-09-06).
Deferred until after the column/row layout is settled; a different mechanism needed.

## Constraints

* Must follow `CAD_STANDARDS.md` (mm units, manifold geometry, centered on the print bed,
  $36–$45 price target, 2.0–3.0mm default wall thickness, Silk-filament-friendly aesthetic).
* Printable without supports.
* Countertop use — freestanding, not wall-mounted.
* Card: 60mm wide (short edge, rests in the slot) × 90mm tall (standing height, "long ways"
  upright). **Fixed — not a design variable.**
* Column gap, row/column counts, tier offsets, tile width, and margins are the **adjustable
  levers** — make the layout work around the fixed 60×90mm card, not the other way around.
* Slot width `SlotWidth`=60.2mm (60mm card + 0.2mm total clearance) — set.
* `NumColumns`=3 (per tile), now correctly driving the column pattern (was previously a
  disconnected Param — fixed 2026-09-06).
* `RowPitch`=35mm is a **provisional** placeholder, chosen only so the row pattern doesn't
  cut past the physical extent of the ramp (the ramp's own diagonal is only ~150mm long; 3
  gaps at the initially-tried 60mm each would have overshot it). **Not a final design
  value** — see open questions.

## Open questions (need Bradley's input before the design is finalized)

* **Interlock mechanism (reopened)** — snap-tab (friction peg/pocket) didn't hold well
  enough. Needs an actual retention mechanism (positive mechanical lock, not just friction)
  — options to consider: a flexing cantilever hook (fatigue risk across repeated
  shows, per the original reason friction was chosen instead), a bolt/screw + insert,
  interlocking dovetail keys, magnets, or something else entirely. Revisit after
  columns/rows below are settled.
* **Row pitch / tier offset — is 35mm (or any single flat number) even the right model?**
  Currently all 4 rows sit on **one continuous ramp** (one long slope, 4 slot-cuts made into
  it at increasing height/depth via `RowPitch`). This may not be the intended physical
  design — a true "theater seating" cascade usually has **each row as a separate discrete
  step** (its own short ramp segment), not 4 cuts into one long slope. Confirm which: (a)
  keep one continuous ramp with rows cut into it at intervals (current implementation), or
  (b) rebuild as 4 distinct stepped tiers. (b) is a bigger rework.
* **Wall/end margins** — how much material stays outside the outermost card column on each
  tile edge.
* Any branding/engraving on the base, per the Silk-filament aesthetic guidance in
  `CAD_STANDARDS.md`?
