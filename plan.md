# Plan — Display Jewelry Cards

**Status: DRAFT — not yet approved.** Columns × rows are now modeled and working (3×4,
audit-clean, survives a `BackHeight` round-trip test). The interlock mechanism is back to
open (snap-tab tried, rejected, removed). Remaining open questions are in `intent.md`.

---

## PARAMETERS

Current, in `Params.FCStd` `VarSet` (all bound where used, audit clean as of 2026-09-06):

| Name | Type | Value | Bound to | Concept |
|---|---|---|---|---|
| `Width` | Length | 200mm | `Sketch.Constraints[10]`, `Pad001.Length`, `Pocket.Length` | Tile width |
| `Depth` | Length | 150mm | `Sketch.Constraints[11]` | Base footprint depth (Y) |
| `LowerBackHeight` | Length | 20mm | `Pad.Length` | Base block height (Z) before ramp |
| `BackHeight` | Length | 30mm | `Sketch001.Constraints[4]` | Ramp's rise |
| `ShelfDepth` | Length | 15mm | `Sketch003` constraints | Retention-notch horizontal run |
| `SlotDepth` | Length | 10mm | `Pocket001.Length` | Card slot cut depth |
| `SlotHeight` | Length | 2mm | `Sketch004.Constraints[9]` | Card slot opening height |
| `SlotWidth` | Length | 60.2mm | `Sketch004.Constraints[8]` | Card slot width (card + clearance) |
| `SlotSpacing` | Length | 3mm | `Sketch001.Constraints[6]`, `Sketch004.Constraints[10-11]` | Shared margin concept (legitimate reuse) |
| `NumColumns` | Integer | 3 | `LinearPattern.Occurrences` | Columns per tile — **wired up 2026-09-06**, was previously dead |
| `RowPitch` | Length | 35mm | `RowPattern.Offset` | Along-slope row spacing — **provisional** |
| `RowCount` | Integer | 4 | `RowPattern.Occurrences` | Rows (tiers) |
| `SnapTabWidth`/`Height`/`Length`/`Clearance` | Length | 14/20/8/0.15mm | — (unused) | Interlock — parked, not deleted |

---

## Fixed vs. adjustable

Card dimensions (60×90mm, standing long-ways) are fixed. Column gap, column count, row
count, row pitch/tier geometry, tile width, and margins are all adjustable.

---

## Column/row pattern — done, with a caveat

Modeled 2026-09-06, audit-clean, `validate_document` all 19 objects healthy, survives a
`BackHeight` 30→45→30 round-trip test without breaking:

- One card slot (`Sketch004`/`Pocket001`) is cut into the shelf's flat retention-notch
  surface, then patterned:
  - **Columns**: `LinearPattern`, `Original`=`Pocket001`, `Direction`=`Sketch004.H_Axis`,
    `Offset`=`SlotWidth+SlotSpacing*2`, `Occurrences`=`NumColumns`.
  - **Rows**: `RowPattern`, `Original`=`Pocket001` (**the plain feature, not the column
    `LinearPattern`**), `BaseFeature`=`LinearPattern` (explicitly chained onto the
    column-patterned tip), `Direction`=`(Sketch001,['Edge1'])` (the ramp's own diagonal —
    stays correct at whatever angle `BackHeight` currently gives), `Offset`=`RowPitch`,
    `Occurrences`=`RowCount`.
- **FreeCAD limitation found:** a `LinearPattern` cannot take another `LinearPattern` as its
  `Originals` (confirmed via `Standard_NullObject NULL shape` at trivial scale — not a
  size/geometry issue). The `Originals`=plain-feature + `BaseFeature`=patterned-tip
  workaround above is what actually works. `PartDesign::MultiTransform` is the "proper"
  FreeCAD mechanism for this and wasn't needed once the workaround was found.
- **Caveat (open question in `intent.md`):** all 4 rows currently cut into **one continuous
  ramp** at 35mm intervals. This may not be the intended final design — true cascading
  tiers usually means 4 **separate physical steps**, not 4 cuts into one long slope. `35mm`
  was chosen only to fit the pattern within the ramp's ~150mm actual length without
  overshooting it (the first value tried, 60mm, would have cut 3 rows past the ramp's
  physical extent). Confirm the real design before treating this as final.

Also fixed while working on this (both were silent parametric gaps `audit_parametric.py`
does **not** check — see `CLAUDE.md` hard rule 1):
- `Sketch003`("Shelf")'s `AttachmentSupport` had gone empty (frozen, non-tracking
  placement) — rebuilt with geometry computed directly from the ramp's line equation
  (`Yb`/`Zt`/`h` formulas in `CLAUDE.md`), verified to track `BackHeight` correctly.
- `Sketch004`("Slot")'s `AttachmentOffset.Base.z` was a bare literal (47) approximating but
  not equal to the shelf's real height (46.94) — bound to the exact same expression.
- The column pattern's `Occurrences` was a plain `3`, with `NumColumns` sitting unused
  alongside it — bound.
- Deleted an orphaned `DatumPlane` that was attached to `Pocket.Face12` (a feature face —
  exactly hard rule 3's warning) and broke the moment `Sketch003` was rebuilt. Confirms the
  rule isn't theoretical for this project.

---

## Snap-tab interlock — built, then rejected

Modeled 2026-09-06: friction-fit peg (`SnapTabPad`) + pocket (`SnapCatchPocket`), verified
geometrically correct (volume-diff matched expected cut volume to 3 decimals), datum-
attached, fully parametric. **Bradley determined it won't hold the two tiles together** and
deleted it the same day. The `SnapTab*` Params remain in `Params.FCStd`, currently unused —
parked for a redesign, not deleted. **Do not resume this until columns/rows (above) are
finalized**, per Bradley's explicit sequencing.

---

## FEATURE TREE (current state)

1. `Sketch`("BoxFooter", `Width`×`Depth`) → `Pad`(`LowerBackHeight`). **Done.**
2. `Sketch001`("SideProfile", ramp profile, `BackHeight`/`SlotSpacing`) → `Pad001`("AngledTop",
   `Width`). **Done.**
3. `Sketch003`("Shelf", retention notch, computed from the ramp equation) → `Pocket`
   (`Width`). **Done, rebuilt 2026-09-06.**
4. `Sketch004`("Slot", one card slot, `SlotWidth`/`SlotHeight`/`SlotSpacing`, Z-position
   bound to the shelf's height) → `Pocket001`(`SlotDepth`). **Done.**
5. `LinearPattern` (columns, `NumColumns`). **Done.**
6. `RowPattern` (rows, `RowPitch`/`RowCount`, chained onto the column pattern). **Done,
   `RowPitch` provisional — see caveat above.**
7. Resolve the "one continuous ramp vs. discrete stepped tiers" open question — may require
   reworking steps 2-6 if the answer is "discrete tiers."
8. Resolve the interlock mechanism (parked).
9. Fillet/chamfer slot mouths per `CAD_STANDARDS.md` aesthetic guidance.
10. Export STL/3MF to `stl/`/`3mf/` once print-ready.

---

## VALIDATION

* `python3 scripts/audit_parametric.py` must report 0 issues. **Currently clean** — but see
  `CLAUDE.md`'s "what it does NOT catch" note; it missed 3 real gaps this session
  (`AttachmentOffset` literal, dead `NumColumns`, empty `AttachmentSupport`). Manually check
  new attachments/patterns' `ExpressionEngine`, don't rely on the script alone.
* `validate_document()`/`validate_object()` after any structural change.
* For any new Pocket: verify cut direction/volume by diffing `Shape.Volume` before/after.
* Bounding box must fit the Creality K2 Plus / ELEGOO Saturn 4 print volumes — the
  120×140mm figure in `CAD_STANDARDS.md` doesn't apply to this design's tiling; verify
  against real bed specs before finalizing tile size.
* Two tiles must physically interlock once a mechanism is chosen — check by placing two
  `App::Link` copies in one assembly, offset by `Width`.
* Print-verify: one test print in PLA before committing to `SlotHeight`/`SlotWidth`/
  `RowPitch`/interlock-clearance final values.

---

## Execution gate

Per `PROJECT_BOOTSTRAP.md`: execution proceeds via Bradley's live iteration in FreeCAD, with
each change verified (audit + `validate_document` + a parametric round-trip test) and
recorded here as it happens — this plan tracks and formalizes that work rather than gating
it ahead of time.
