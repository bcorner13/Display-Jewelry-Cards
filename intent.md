# Intent

## Goal

A countertop tiered display rack for 60×90mm earring cards, standing upright on their short
(60mm) edge. Cards are arranged in a **5-column × 4-row grid** (20 cards per full display).
Rows are **tiered/cascading** — each row sits higher and further back than the one in front
(like theater seating), so every row's cards stay visible from the front. This matches the
angled-backrest shelf already modeled (`SideProfile`/`Pad001`).

The full 5×4 assembly is built as **two mirror-symmetric half-tiles** (~210mm wide each,
split down the center column) rather than one 420mm-wide plate, so each printed piece stays
small enough to transport. (Confirmed 2026-09-06 — supersedes the 120×140mm figure in
`CAD_STANDARDS.md`, which does not apply to this design's tiling decision.) `Base.FCStd`
currently models **one tile** — `Width` (210mm) is that tile's width, not the full assembly's.

## Constraints

* Must follow `CAD_STANDARDS.md` (mm units, manifold geometry, centered on the print bed,
  $36–$45 price target, 2.0–3.0mm default wall thickness, Silk-filament-friendly aesthetic).
* Printable without supports.
* Countertop use — freestanding, not wall-mounted.
* Card: 60mm wide (short edge, rests in the slot) × 90mm tall (standing height, "long ways"
  upright). **Fixed — not a design variable.** Confirmed 2026-09-06.
* Column spacing, row/column counts, tile split, and margins are the **adjustable levers** —
  make the layout work around the fixed 60×90mm card, not the other way around.
* Column spacing: ~16mm gap between adjacent cards → ~76mm column pitch (60mm card + 16mm
  gap), edge to edge. **Not yet built into the model** — the existing shelf (`Pad001`) is one
  continuous full-width ridge with no per-column dividers yet.
* `SlotSpacing`(3mm)/`SlotHeight`(2mm)/`SlotDepth`(25mm)/`BackHeight`(30mm) govern the
  Y-Z retention-lip cross-section (how a card's top-back edge is held) — a **different
  concept** from the 16mm column-to-column gap. Do not reuse `SlotSpacing` for column pitch;
  name that separately (see `plan.md`).
* Eventually may add a lid to house cards in their slots for transport — deferred, not
  blocking current work.

## Open questions (need Bradley's input before `plan.md` is finalized)

* **Interlock mechanism** between the two half-tiles — tongue-and-groove, dowel pins,
  printed-in snap features? Not yet decided.
* **Tier offset** — how far back/up each successive row shifts relative to the one in front.
  Depends on the backrest lean angle and the 90mm card height; not yet computed against real
  numbers (current `SideProfile` values look like an early rough pass, not final).
* **Wall/end margins** — how much material stays outside the outermost card column on each
  tile edge (affects strength + interlock clearance).
* Any branding/engraving on the base (e.g. a logo or shop name), per the Silk-filament
  aesthetic guidance in `CAD_STANDARDS.md`?
