# Plan — Display Jewelry Cards

**Status: DRAFT — not yet approved.** Depends on the remaining open questions in `intent.md`
(interlock mechanism, tier offset, wall margins). `Base.FCStd` now models **one half-tile**
of the eventual two-tile, 5×4-card assembly (see `intent.md`). The base block + one shelf
ridge are fully parametric; the actual per-column card slots don't exist yet — the ridge is
still one continuous full-width feature.

---

## PARAMETERS

Existing, in `Params.FCStd` `VarSet` (all bound, audit clean as of 2026-09-06):

| Name | Type | Value | Bound to | Concept |
|---|---|---|---|---|
| `Width` | `App::PropertyLength` | 210mm | `Sketch.Constraints[10]`, `Pad001.Length` | **This tile's** width (half of the 420mm full-assembly span) |
| `Depth` | `App::PropertyLength` | 150mm | `Sketch.Constraints[11]` | Base footprint depth (Y) |
| `LowerBackHeight` | `App::PropertyLength` | 50mm | `Pad.Length` | Base block height (Z) before the ridge |
| `BackHeight` | `App::PropertyLength` | 30mm | `Sketch001.Constraints[4]` | Extra ridge height above the base |
| `SlotDepth` | `App::PropertyLength` | 25mm | `Sketch001.Constraints[8]` | Retention-lip shelf run (Y-Z profile) |
| `SlotHeight` | `App::PropertyLength` | 2mm | `Sketch001.Constraints[13]` | Retention-lip notch height (Y-Z profile) |
| `SlotSpacing` | `App::PropertyLength` | 3mm | `Sketch001.Constraints[6]` | Retention-lip notch step depth (Y-Z profile) |

**Naming trap to avoid:** `SlotSpacing`/`SlotHeight`/`SlotDepth` describe the *retention-lip
cross-section* (how a card's top-back edge is caught — a Y-Z profile, extruded the same
across the whole width). They are **not** the 16mm column-to-column gap. Do not repurpose
them for column pitch — add distinct variables (below).

To add, once the open questions in `intent.md` are answered:

| Name | Type | Proposed value | Purpose |
|---|---|---|---|
| `CardWidth` | `App::PropertyLength` | 60mm | Card short-edge width (confirmed 2026-09-06) |
| `CardHeight` | `App::PropertyLength` | 90mm | Card standing height (confirmed 2026-09-06) |
| `ColumnGap` | `App::PropertyLength` | 16mm | Gap between adjacent card slots, edge to edge (confirmed 2026-09-06) — column pitch = `CardWidth + ColumnGap` = 76mm |
| `ColumnCount` | `App::PropertyInteger` | 5 (full assembly) | Columns in the full 5×4 grid — **per-tile count is not a clean integer** (5 split across 2 tiles down the center column); model the tile from absolute column positions relative to tile center, not a per-tile linear-pattern count |
| `RowCount` | `App::PropertyInteger` | 4 | Rows (tiers) in the full grid |
| `RowRise` / `RowSetback` | `App::PropertyLength` | TBD | Per-row Z rise / Y setback for the cascading tiers — depends on the answer to the "tier offset" open question |
| `CardSlotClearance` | `App::PropertyLength` | TBD | Print-fit slack added to actual card thickness — **do not reuse `SlotHeight`** (that's the existing lip's raw opening spec, a different interface) |

---

## Fixed vs. adjustable

Confirmed 2026-09-06: **card dimensions (60×90mm, standing long-ways) are fixed** — not a
design variable. `CardWidth`/`CardHeight` above are locked. Everything else — column count,
column gap, row count, tier offsets, tile split point, margins — is adjustable to make the
5×4 layout work around that fixed card size.

(A "Sample Card" reference body — three rectangles at a rough, non-final pitch, made to
confirm card orientation — existed briefly and was deleted 2026-09-06 once its job was
done. Not present in the model.)

---

## FEATURE TREE (ordered)

1. **Existing, fully parametric** — `Sketch` ("BoxFooter", `Width`×`Depth`) → `Pad` (base
   block, `LowerBackHeight`) → `Sketch001` ("SideProfile", full-width retention-lip profile,
   `BackHeight`/`SlotHeight`/`SlotSpacing`/`SlotDepth`) → `Pad001` (ridge, fused across
   `Width`). This is the **top row's** shelf.
2. Resolve remaining `intent.md` open questions (interlock mechanism, tier offset, wall
   margins) — needed before step 3 can be dimensioned correctly.
3. Add `CardWidth`, `CardHeight`, `ColumnGap`, `ColumnCount`, `RowCount` to `Params.FCStd`.
4. Divide the existing continuous ridge into individual card slots: sketch the per-column
   dividers/pockets on a datum-plane-attached sketch (not a feature face — hard rule 3),
   positioned from `CardWidth`/`ColumnGap`, accounting for the center column being split
   across the two tiles.
5. Add 3 more tiered rows above/behind the first, each a repeat of the row-1 shelf
   (`Sketch001`/`Pad001` pattern) offset by `RowRise`/`RowSetback` — confirm whether a
   `linear_pattern`/`polar_pattern` fits this (uniform per-row offset) or each row needs its
   own sketch (non-uniform cascading geometry).
6. Design and add the interlock feature between the two tiles, per the chosen mechanism.
7. Re-run `python3 scripts/audit_parametric.py` — must be clean.
8. Fillet/chamfer slot mouths per `CAD_STANDARDS.md` aesthetic guidance (avoid sharp
   overhangs for Silk filament).
9. Export STL/3MF to `stl/`/`3mf/` once print-ready — export **both** mirrored tiles.

---

## CONSTRAINT STRATEGY

* Base rectangle (`Sketch`/"BoxFooter") stays fully constrained via `Width`/`Depth` —
  already done, and already symmetric about the Y-axis (`Symmetric` constraint present from
  the start), so the tile stays centered as `Width` changes.
* New slot/column sketches attach to `PartDesign::Plane` datums offset from the base top
  face — never to `Pad`/`Pocket`/`Body` faces directly (hard rule 3).
* Every dimensional constraint binds to a Params variable at creation time — no interim
  literals, even temporarily, per hard rule 1.
* Column layout comes from `CardWidth`/`ColumnGap` math, not hand-placed per-slot numbers —
  keeps the pitch a single source of truth across both tiles.
* The two tiles should be **one parametric definition, mirrored** (e.g. a `Mirror` feature
  or a shared macro run twice with a mirror flag) rather than two independently-maintained
  files — avoids the two halves drifting out of sync.

---

## VALIDATION

* `python3 scripts/audit_parametric.py` must report 0 issues before any commit.
* Manifold check via `validate_object`/`validate_document` (or `part_check_shape`) after the
  slot pocket + pattern are added.
* Bounding box must fit the Creality K2 Plus / ELEGOO Saturn 4 print volumes — confirm the
  210mm tile width and full tile footprint against actual bed dimensions (the 120×140mm
  figure in `CAD_STANDARDS.md` was confirmed 2026-09-06 to not apply to this design's
  tiling decision — verify against the real bed specs before finalizing tile size).
* Two tiles must physically interlock and reproduce a flush, structurally sound center
  column when joined — check by assembling both in one document/assembly.
* Print-verify: one test print in PLA before committing to `SlotHeight`/`CardSlotClearance`/
  tier-offset final values (per the global workflow — tune after test print, no rework).

---

## Execution gate

Per `PROJECT_BOOTSTRAP.md`: **execution is forbidden until this plan is approved.** Steps
2–9 above touch the FreeCAD model and must not proceed until Bradley confirms:
(a) the remaining open questions in `intent.md`, and (b) this plan.

(Note: Bradley has been iterating directly in FreeCAD throughout this bootstrap — this plan
tracks and formalizes that live work rather than gating it; treat each confirmed change as
approved once made and verified, per the running conversation.)
