# Plan — Display Jewelry Cards

**Status: DRAFT — core geometry done, verification pending.** Row/column layout (3×7,
discrete stepped tiers) and the connector-strip interlock are both modeled and audit-clean.
Remaining work is verification (print test, assembly fit-check) and cosmetic finishing.

---

## PARAMETERS

See `CLAUDE.md`'s "Params variables" table for the full current list (grouped: Base
geometry, Shelf/slot, Pattern, Connector, Snap-tab-parked). All bound, audit clean as of
2026-09-07.

---

## Row/column layout — done

3 columns × 7 rows (21 slots), built as **real discrete stepped tiers**:
- `LinearPattern001` repeats the shelf notch itself 7× along the ramp (`Mode=Extent`,
  `Offset=Depth`) — creates 7 actual physical steps.
- `RowPattern` repeats the card slot 7× to match (`Mode=Extent`, `Offset=RowPitch`),
  chained onto the column pattern's tip.
- Verified by volume math (exact match) and visually (screenshot: 7 clean steps, 3 slot
  grooves each).
- See `CLAUDE.md` hard rules 3, 6, 7 for the debugging history (sign-convention bug,
  stuck-object NULL-shape bug, `ObjectXZ`-retrofit unreliability) — worth reading before
  touching this part of the tree again.

## Connector-strip interlock — done

Replaces the rejected snap-tab (see `intent.md`). `ConnectorStrip.FCStd`: capsule bridge,
pin at each end (`ConnectorPinSpan`=30mm apart, `ConnectorRadius`=5mm,
`ConnectorPinHeight`=5mm, chamfered). `Base.FCStd`: two "keyhole" pockets (deep round pin
hole + shallow rectangular plate recess for a flush fit), 15mm inset (`ConnectorInset`)
from each tile edge, left and right — symmetric, so any tile works as leftmost/middle/
rightmost. Verified by volume diff (exact match to the computed expected value) and
visually (bottom-view screenshot).

---

## FEATURE TREE (current state)

1-6: Base block, ramp, shelf notch (patterned ×7 as steps), card slot (patterned ×3
    columns ×7 rows = 21). **Done** — see `CLAUDE.md` Assembly architecture for the exact
    object names/chain.
7. Connector pockets (left + right, pin hole + plate recess). **Done.**
8. Populate `Assembly.FCStd` — place two tile copies + a `ConnectorStrip`, add joints,
   verify physical fit (pin seats fully, no interference, tiles sit flush at the seam).
9. Fillet/chamfer slot mouths and step edges per `CAD_STANDARDS.md` aesthetic guidance
   (avoid sharp overhangs for Silk filament).
10. Test print (PLA first, per project convention) — verify `SlotHeight`/`SlotWidth`/
    `ConnectorClearance` actually fit real cards and the real connector before treating
    them as final.
11. Export STL/3MF to `stl/`/`3mf/` — one tile file + one connector file (each tile uses
    the same STL, printed twice).

---

## VALIDATION

* `python3 scripts/audit_parametric.py` must report 0 issues. **Currently clean across all
  4 project files.** Remember what it doesn't catch — see `CLAUDE.md` hard rule 1.
* `validate_document()`/`validate_object()` after any structural change (ignore the sketch
  "non-positive volume" false positive — hard rule 9).
* For any new Pocket: verify cut direction/volume by diffing `Shape.Volume` before/after.
* Bounding box must fit the Creality K2 Plus / ELEGOO Saturn 4 print volumes — the
  120×140mm figure in `CAD_STANDARDS.md` doesn't apply to this design's tiling; verify
  against real bed specs before finalizing tile size.
* **Assembly fit-check** (not yet done): populate `Assembly.FCStd` with two tiles +
  connector, confirm the pin fully seats and the tiles sit flush at the seam.
* Print-verify: one test print in PLA before committing to `SlotHeight`/`SlotWidth`/
  `RowPitch`/`ConnectorClearance` final values.

---

## Execution gate

Per `PROJECT_BOOTSTRAP.md`: execution proceeds via Bradley's live iteration in FreeCAD, with
each change verified (audit + `validate_document` + volume/geometry checks) and recorded
here as it happens — this plan tracks and formalizes that work rather than gating it ahead
of time.
