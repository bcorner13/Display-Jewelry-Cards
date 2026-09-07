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
`ConnectorPinHeight`=5mm, chamfered). `Base.FCStd`: **2 pairs of "keyhole" pockets per
side** (4 total — Bradley wanted 2 per side for torsional rigidity), each pair a deep round
pin hole + shallow rectangular plate recess for a flush fit, 15mm inset (`ConnectorInset`)
from each tile edge, symmetric front/back about `Y=0` via `ConnectorPairOffset`(50mm) — so
any tile works as leftmost/middle/rightmost. Verified by volume diff (exact match to the
computed expected value, 2nd pair an exact 2× multiple of the 1st) and visually
(bottom-view screenshot: 4 clean gap-free pockets).

**Fixed a real bug post-first-pass:** the plate recess was originally built tangent to the
pin hole's *edge*, not overlapping it — left an uncut crescent-shaped sliver everywhere
except the single tangent point (looked like a visible gap in a render, confirmed by
point-containment test). Fixed by starting the rectangle at the pin's *center* X instead —
see `CLAUDE.md` hard rule 11.

**Also hit a live data-loss incident**: all 7 `Connector*` Params vanished mid-session
(from both the live document and disk) while everything else survived — see `CLAUDE.md`
hard rule 10 for the symptom (expressions silently frozen, error only in Report View) and
recovery (re-add the same properties; downstream expressions re-resolve automatically).

---

## FEATURE TREE (current state)

1-6: Base block, ramp, shelf notch (patterned ×7 as steps), card slot (patterned ×3
    columns ×7 rows = 21). **Done** — see `CLAUDE.md` Assembly architecture for the exact
    object names/chain.
7. Connector pockets, **2 pairs per side** (front + back, torsional rigidity). **Done.**
8. Populate `Assembly.FCStd` — two tile `App::Link`s + two `ConnectorStrip` `App::Link`s
   (front/back seam pair). **Done** (2026-09-07) — plain fixed-`Placement` links, not
   solved joints, but positioned by the same math as the pocket geometry and verified by
   Boolean intersection (0mm³ interference on all 4 pin/pocket pairs) — see `CLAUDE.md`.
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
* **Assembly fit-check — done** (2026-09-07): `Assembly.FCStd` has two tiles + two
  connectors positioned via the same X/Y math as the pocket geometry; confirmed 0mm³
  Boolean interference on all 4 pin/pocket pairs and edge-to-edge tile alignment (no gap,
  no overlap). Not joint-solved (plain fixed links) — sufficient for a geometric fit-check,
  not a substitute for the physical print-verify step below.
* Print-verify: one test print in PLA before committing to `SlotHeight`/`SlotWidth`/
  `RowPitch`/`ConnectorClearance` final values.

---

## Execution gate

Per `PROJECT_BOOTSTRAP.md`: execution proceeds via Bradley's live iteration in FreeCAD, with
each change verified (audit + `validate_document` + volume/geometry checks) and recorded
here as it happens — this plan tracks and formalizes that work rather than gating it ahead
of time.
