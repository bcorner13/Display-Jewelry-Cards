# Plan — Display Jewelry Cards

**Status: DRAFT — not yet approved.** Depends on answers to the open questions in
`intent.md` (card size, capacity, slot pitch, base height). The parameters and feature tree
below are written against the *existing* geometry (base block + one ridge, no slots cut yet)
plus the one remaining parametric gap (`Sketch001.Constraints[4]` was fixed by Bradley on
2026-09-06 — see `BackHeight` below). Update this plan once the open questions come in, then
get explicit approval before any further FreeCAD execution.

---

## PARAMETERS

Existing, in `Params.FCStd` `VarSet`:

| Name | Type | Default | Status |
|---|---|---|---|
| `Width` | `App::PropertyLength` | 420mm | ✅ bound (`Sketch.Constraints[10]`, `Pad001.Length`) |
| `Depth` | `App::PropertyLength` | 150mm | ✅ bound (`Sketch.Constraints[11]`) |
| `SlotDepth` | `App::PropertyLength` | 25mm | ✅ bound (`Sketch001.Constraints[8]`) |
| `SlotHeight` | `App::PropertyLength` | 2mm | ✅ bound (`Sketch001.Constraints[13]`) |
| `SlotSpacing` | `App::PropertyLength` | 3mm | ✅ bound (`Sketch001.Constraints[6]`) |
| `BackHeight` | `App::PropertyLength` | 30mm | ✅ bound (`Sketch001.Constraints[4]`) — added by Bradley 2026-09-06, fixing the audit finding on this constraint |

To add (fixes the 1 remaining audit finding):

| Name | Type | Default | Purpose |
|---|---|---|---|
| `BaseHeight` | `App::PropertyLength` | 50mm | Binds `Pad.Length` — overall base block height (Z) before the ridge feature |

To add once slot-cutting is designed (pending `intent.md` answers):

| Name | Type | Purpose |
|---|---|---|
| `SlotCount` | `App::PropertyInteger` | Number of card slots — drives the linear pattern |
| `CardSlotClearance` | `App::PropertyLength` | Print-fit slack added to the card's actual thickness — **do not reuse `SlotHeight`** for this (hard rule 4/5: `SlotHeight` is the raw opening spec, clearance is a distinct concept) |
| `WallBetweenSlots` | `App::PropertyLength` | Material thickness of each comb tooth between slots |

---

## FEATURE TREE (ordered)

1. **Existing** — `Sketch` (`Width`×`Depth` rectangle) → `Pad` (base block, height currently
   unbound) → `Sketch001` (ridge profile, fully bound) → `Pad001` (ridge, fused).
2. Add `BaseHeight` to `Params.FCStd`; bind `Pad.Length` to it via a
   `macros/bind_base_height.FCMacro`.
3. Confirm card size/capacity (open questions in `intent.md`) → add `SlotCount`,
   `CardSlotClearance`, `WallBetweenSlots` to `Params.FCStd`.
4. Model one slot as a `PartDesign::Pocket` on a datum-plane-attached sketch (not a feature
   face — hard rule 3), dimensioned from `SlotHeight`, `SlotDepth`, `CardSlotClearance`.
5. `linear_pattern` the slot `SlotCount` times across `Width`, spaced by `SlotSpacing` +
   `WallBetweenSlots`.
6. Re-run `python3 scripts/audit_parametric.py` — must be clean.
7. Fillet/chamfer slot mouths per `CAD_STANDARDS.md` aesthetic guidance (avoid sharp overhangs
   for Silk filament).
8. Export STL/3MF to `stl/`/`3mf/` once print-ready.

---

## CONSTRAINT STRATEGY

* Base rectangle (`Sketch`) stays fully constrained via `Width`/`Depth` — already done.
* New slot sketch(es) attach to a `PartDesign::Plane` datum offset from the base top face —
  never to `Pad`/`Pocket`/`Body` faces directly (hard rule 3).
* Every dimensional constraint binds to a Params variable at creation time — no interim
  literals, even temporarily, per hard rule 1.
* Slot count and spacing come from one `linear_pattern` off a single fully-constrained slot
  sketch, not N hand-drawn slots — keeps `SlotCount` a single source of truth.

---

## VALIDATION

* `python3 scripts/audit_parametric.py` must report 0 issues before any commit (current
  baseline: 1 issue, tracked above as step 2).
* Manifold check via `validate_object`/`validate_document` (or `part_check_shape`) after the
  slot pocket + pattern are added.
* Bounding box must fit the Creality K2 Plus / ELEGOO Saturn 4 beds per `CAD_STANDARDS.md` —
  check once `Width`/`Depth`/`BaseHeight` are finalized (420mm width is worth double-checking
  against actual bed size before treating it as final).
* Print-verify: one test print in PLA before committing to `SlotHeight`/`CardSlotClearance`
  final values (per the global workflow — tune after test print, no rework).

---

## Execution gate

Per `PROJECT_BOOTSTRAP.md`: **execution is forbidden until this plan is approved.** Steps 2–8
above touch the FreeCAD model and must not proceed until Bradley confirms:
(a) the open questions in `intent.md`, and (b) this plan.
