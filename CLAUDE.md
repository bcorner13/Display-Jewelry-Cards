# Project rules — Display Jewelry Cards

This is a countertop **tiered** display rack for 60×90mm earring cards standing upright
long-ways on their short edge — **card dimensions are fixed**; row/column counts, spacing,
and the tile split are the adjustable levers. Built as **two identical tiles** (same part
printed twice, not mirrored), joined underneath by a separate **connector strip**
(`ConnectorStrip.FCStd`) with pins that seat in matching pockets cut into each tile's
bottom — **2 pockets per side** (4 total: left-front, left-back, right-front, right-back,
symmetric about `Y=0` via `ConnectorPairOffset`), for torsional rigidity. As of 2026-09-07:
**3 columns × 7 rows** of card slots, cut as actual **discrete stepped tiers** (a real
staircase — Bradley's redesign, not a continuous ramp with cuts at intervals).
`audit_parametric.py` clean across all 4 project files (`Base.FCStd`, `Params.FCStd`,
`ConnectorStrip.FCStd`, `Assembly.FCStd`).

**A live-session data-loss incident happened 2026-09-07**: all 7 `Connector*` VarSet
properties vanished from the live document (and the saved file) partway through this
session, while everything else (row/column Params, `Width`, etc.) survived intact —
mechanism not confirmed, but every expression referencing them kept its formula string and
just failed to evaluate (`Property 'ConnectorInset' not found...`), silently freezing at
the last good value rather than erroring loudly in the 3D view. **If any `Connector*`-named
constraint looks stuck at an old value, check the Report View for "not found" errors before
assuming your expression edit didn't take** — see hard rule 10.

> **How to use this file:** every `[FILL: …]` marker from the bootstrap template has been
> filled from direct inspection (MCP + on-disk XML) — not guessed. Where something is
> inferred rather than confirmed, it's labeled below.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** Currently clean, but this project has repeatedly hit gaps
   `audit_parametric.py` doesn't catch — treat it as necessary, not sufficient:
   - A sketch's `AttachmentSupport` silently going empty (frozen placement, stops tracking
     anything) — the script only checks for *face*-attachment, not *no* attachment.
   - `AttachmentOffset` literals (e.g. a bare `47` positioning a sketch) — the script only
     checks sketch *constraints* and Pad/Pocket/Chamfer/Fillet properties.
   - `LinearPattern`/`PolarPattern` `Occurrences`/`Offset` left unbound while a same-named
     Param sits nearby unused (`NumColumns` existed for a session before anything read it).
   - **A VarSet's own property drifting from a formula used elsewhere.** `Width` was an
     independent stored value (200mm) while `BoxFooter`'s own footprint constraint was
     derived (`NumColumns*(SlotWidth+2*SlotSpacing)` = 198.6mm) — a 1.4mm silent mismatch
     between the footprint and the ramp/pocket extrusions built on `Width`. Fixed by binding
     `Width` itself to that same formula (a VarSet property can reference its own siblings
     directly, no `#VarSet.` prefix needed for same-object references).
   - **Lesson: after adding any new attachment, pattern, or cross-referencing Param, manually
     check its `ExpressionEngine` and compare any two Params that should stay in sync** —
     don't rely on the audit script alone.

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this
   project; rule applies preventively.

3. **Attach sketches to datum planes, not feature faces.** Hit this failure mode directly
   once (an orphaned `DatumPlane` attached to `Pocket.Face12` broke the moment upstream
   geometry changed) — deleted it, unused by anything real. **Also learned:** retrofitting
   `MapMode='ObjectXZ'` onto an *existing* sketch (rather than one created fresh with that
   mode) does not reliably recompute — `Sketch003`'s `Placement` stayed frozen at its old
   value even with `AttachmentSupport=[]` and a fresh identity `AttachmentOffset`. Fixed by
   switching to an explicit `YZ_Plane` + `MapMode='FlatFace'` attachment instead — this
   pattern has proven reliable throughout the project; treat `ObjectXZ` retrofits on
   existing sketches as unreliable.

4. **Clearance concepts stay decoupled — now 3 distinct mating interfaces:**
   - `SlotWidth` (60.2mm) — card-to-slot print fit.
   - `SnapTabClearance` (0.15mm/side) — **unused**, parked with the rejected snap-tab design.
   - `ConnectorClearance` (0.2mm/side) — connector-pin-to-pocket print fit. Own dedicated
     Param; do not merge with the other two.

5. **One knob, one concern — one deliberate reuse.** `SlotSpacing` (3mm) drives both
   `SideProfile`'s top-cap segment and `Slot`'s side margins — same real concept (a
   consistent small margin), treated as legitimate reuse, not a collision. If they ever need
   to diverge, split it.

6. **Coordinate sign conventions differ by attachment mode — verify, don't assume.**
   `Sketch001`'s `ObjectXZ` mode and a `YZ_Plane`/`FlatFace` sketch map local-X to global-Y
   with **opposite signs** (`Sketch001` local `(75,20)`→global `Y=-75`; a `YZ_Plane` sketch's
   local `x=+7`→global `Y=+7`). Copying coordinate *values* from one sketch's local frame
   into a different sketch attached a different way — even to "the same plane" — silently
   mirrors the geometry. Caused a real bug: a rebuilt `Shelf` notch computed with the wrong
   sign landed entirely in empty air (0mm³ removed, verified valid shape that just cut
   nothing). **Always verify a new sketch's actual global `Placement`/`Shape.BoundBox`
   against a known-good reference point before trusting copied coordinate values.**

7. **A `PartDesign::Pocket`/`Pad` that has had its properties (`BaseFeature`, `Reversed`,
   `Profile`, etc.) reassigned multiple times during interactive debugging can get stuck in
   a bad state that produces `Standard_NullObject … NULL shape` errors even though the
   underlying geometry is completely valid** (confirmed: identical raw `Part.cut()` between
   the same two shapes succeeded every time; only the `PartDesign::Pocket` *feature* wrapping
   it failed). **The reliable fix is delete-and-recreate fresh, not further mutation** — this
   cost significant debugging time twice in this project (once on `Sketch003`'s original
   fix, once on the connector pockets) before the pattern was recognized. When a `PartDesign`
   feature gives a NULL-shape error, don't just toggle `Reversed`/`Length` repeatedly on the
   same object — after 1-2 tries, delete it and rebuild fresh.

8. **PartDesign features created via raw scripting don't auto-manage `ViewObject.Visibility`**
   the way the GUI/typed tools do. After adding a new tip feature, explicitly hide the old
   tip and show the new one (`obj.ViewObject.Visibility`) — otherwise the 3D view keeps
   showing a stale intermediate feature, which looks exactly like "the geometry doesn't
   exist" even though it computed correctly. This caused real confusion (Bradley: "where are
   the steps and slots") when a correct `RowPattern` result existed but an earlier `Pocket`
   was still the visible one.

9. **`validate_document`/`validate_object`'s "non-positive volume" warning on a
   `Sketcher::SketchObject` is a false positive** — sketches are 2D wires/faces with no
   volume by definition. Don't treat it as a real defect; check the downstream solid
   feature's `Shape.isValid()`/`Volume` instead.

10. **A VarSet's dynamic properties can vanish from a live/saved document mid-session**
    (confirmed 2026-09-07 — see the incident note at the top of this file). All 7
    `Connector*` properties disappeared while every other Param survived; expressions
    referencing them kept their formula strings (visible in `ExpressionEngine`) but silently
    failed to evaluate, leaving the constraint frozen at its last successfully-computed
    value with **no error in the FreeCAD Python console** — the error only appears in the
    GUI's Report View (`Property 'X' not found in 'Params#VarSet.X' in property binding
    ...`). **If an expression edit via `setExpression` doesn't change a constraint's live
    `.Value` after a recompute, don't assume the recompute needs forcing again — check
    whether the referenced Param still exists at all** (`"X" in vs.PropertiesList`) before
    re-debugging the recompute mechanics. Fix is straightforward: re-`addProperty` with the
    same name/type/value: everything downstream re-resolves automatically once the property
    exists again, no need to touch the expressions themselves.

11. **A shape that's merely *tangent* to another cut (touching at one point, not
    overlapping) leaves an uncut sliver, not a smooth merge.** A rectangular pocket built
    flush against a circular pocket's outer edge (tangent at one point) left a crescent of
    uncut material everywhere except that single point — looked like a real gap in a
    render, confirmed by point-containment test. **Fix: make adjoining pocket profiles
    overlap by starting the straight-edged piece at the round piece's *center*, not its
    edge** — guarantees no gap regardless of how the two shapes' curvature differs. Applied
    to all 4 (now 8, after the 2nd pair) connector plate-recess sketches: near edge fixed at
    `ConnectorInset` distance (the pin's own center X), not `ConnectorInset ± radius`.

---

## Assembly architecture

Confirmed product intent (see `intent.md`): a countertop rack of 60×90mm earring cards in a
**3-column × 7-row** grid (21 slots/tile), tiered as **real discrete stepped tiers**
(Bradley's redesign — each row is its own physical stair step, not a cut into one
continuous ramp). Two identical tiles join underneath via a separate connector strip.

### `Base.FCStd` (one tile, 28 objects)

- `Body` → `Sketch`("BoxFooter", `XY_Plane`) → `Pad`("BoxFooter001", base block,
  `Length`=`LowerBackHeight`=15mm). Footprint: `Width`(derived, 198.6mm)×`Depth`(150mm).
- `Sketch001`("SideProfile", `MapMode=ObjectXZ`, body-local — created fresh this way, so
  it recomputes reliably; see hard rule 3) → `Pad001`("AngledTop", additive, `Length`=
  `Width`). The angled backrest profile: `BackHeight`/`SlotSpacing`-driven.
- `Sketch003`("Shelf", **rebuilt on `YZ_Plane`/`FlatFace`**, not `ObjectXZ` — see hard
  rule 3) → `Pocket`("Shelf Pocket"): a right-triangle wedge notch at the ramp's back-top
  corner. Corners derived algebraically from the ramp's own line equation (`Yb`/`Zt`/`h` —
  see git history for the exact formulas) — no face-attachment, no rotation, purely
  Params-driven, verified to track `BackHeight` correctly.
- `Sketch004`("Slot", `MapMode=FlatFace`, attached to the **datum** `XY_Plane`) →
  `Pocket001`: one card slot, `SlotWidth`×`SlotHeight`, depth `SlotDepth`. Z-position bound
  to the same expression that drives the Shelf notch's flat step.
- `LinearPattern`("SlotPattern"): `Original`=`Pocket001`, `Direction`=`Sketch004.H_Axis`,
  `Mode=Spacing`, `Offset`=`SlotWidth+SlotSpacing*2`, `Occurrences`=`NumColumns`(3) — the
  3-column repeat.
- `LinearPattern001`: `Original`=`Pocket`(the Shelf notch, **not** `Pocket001`),
  `BaseFeature`=`Pocket001`, `Direction`=`(Sketch001,['Edge1'])` (the ramp's own diagonal
  edge), **`Mode=Extent`**, `Offset`=`Depth`(150mm), `Occurrences`=`RowCount`(7) — repeats
  the **shelf notch itself** 7 times along the ramp, evenly spread across the ramp's ~full
  length, creating 7 actual physical stair steps. (Bradley's fix for the "0 rows physically
  fit" problem I'd hit with a `Spacing`-mode, fixed-per-step-offset approach — `Extent` mode
  spreads occurrences across a *total* span instead, which is what makes 7 steps fit.)
- `RowPattern`: `Original`=`Pocket001`(the single slot, plain feature — **not** a pattern
  object, per the nesting limitation below), `BaseFeature`=`LinearPattern`(chains onto the
  column-patterned tip), `Direction`=`Edge1`, `Mode=Extent`, `Offset`=`RowPitch`(35mm),
  `Occurrences`=`RowCount`(7) — repeats the **slot** 7 times to match the 7 physical steps.
  Verified by volume diff: expanding from 3→21 slots removes exactly 18× one slot's volume.
  - **FreeCAD limitation confirmed:** a `PartDesign::LinearPattern` cannot take another
    `LinearPattern` as its `Originals` (`Standard_NullObject NULL shape`, reproduced at
    trivial scale — structural, not size-related). Workaround used throughout: `Originals`
    = the plain pre-pattern feature, `BaseFeature` explicitly set to the already-patterned
    tip. `PartDesign::MultiTransform` is the "proper" mechanism for this; not needed once
    the workaround was found.
- **Connector pockets** (added 2026-09-07, chained onto `RowPattern`), **2 pairs per side**
  (4 total — Bradley wanted 2 for torsional rigidity, not the 1 pair first built), each a
  "keyhole": a deep round pin hole + a shallow rectangular plate recess so the connector
  sits flush with the tile's bottom. Suffix `2` = the second (back, `Y=-ConnectorPairOffset`)
  pocket in each pair; unsuffixed = the first (front, `Y=+ConnectorPairOffset`):
  - `ConnectorHoleSketchLeft`/`Right`[`2`] → `ConnectorPocketLeft`/`Right`[`2`]: circle,
    radius `ConnectorRadius+ConnectorClearance`, centered at
    `X=∓(Width/2-ConnectorInset)`, `Y=±ConnectorPairOffset`, on `XY_Plane` (the tile's
    bottom face, Z=0). Depth `ConnectorPinHeight+ConnectorClearance`.
  - `ConnectorPlateSketchLeft`/`Right`[`2`] → `ConnectorPlatePocketLeft`/`Right`[`2`]:
    rectangle from the pin hole's **center** X (not its edge — see hard rule 11, this was
    a real bug: tangent-only left an uncut gap) out to the tile's edge, width
    `2×(ConnectorRadius+ConnectorClearance)`, depth `ConnectorBaseThickness+
    ConnectorClearance`.
  - All 8 built as **fresh objects in one pass** (create → constrain → bind, no property
    reassignment afterward) after hitting hard rule 7's NULL-shape issue repeatedly on
    reused/mutated objects. Verified: total volume removed (2 pins + 2 plates per pair,
    ×2 pairs) matches the computed expected value exactly — 2nd pair's removal is an exact
    2× multiple of the 1st pair's.
  - Final tip: `ConnectorPlatePocketRight2`. `Body.Tip` set explicitly.

### `ConnectorStrip.FCStd` (separate file, 15 objects)

A capsule-shaped bridge: `Sketch`(stadium outline, `ConnectorPinSpan`=30mm between arc
centers, `ConnectorRadius`=5mm) → `Pad`(`ConnectorBaseThickness`=1mm) + `Sketch001`(circle,
`Equal`-constrained to the same radius, `Coincident` to the capsule's left arc center) →
`Pad001`(`ConnectorPinHeight`=5mm boss) → `Chamfer`(`ConnectorChamfer`=1mm) →
`Mirrored`(across `Sketch.V_Axis`, duplicating the boss to the right end). Fully bound via
cross-document expressions (`Params#VarSet.ConnectorXxx`) — was **entirely unbound** when
found this session (5 audit findings), fixed by adding the shared `Connector*` Params.

Two identical printed tiles + one `ConnectorStrip` per seam: each pin (15mm from the
connector's center) drops into the matching tile's pocket (15mm from that tile's edge) —
pin-to-pin span (30mm) exactly matches two tiles' combined insets when butted together.

**Snap-tab interlock (tried, then rejected, 2026-09-06):** a friction-fit peg+pocket built
into the tile itself — Bradley determined it wouldn't hold the tiles together. Superseded
by the separate `ConnectorStrip` approach above. `SnapTab*` Params remain, unused — not
deleted, in case revisited.

(A "Sample Card" reference body existed briefly 2026-09-06 to confirm card orientation,
then was deleted. Not present in the model.)

### `Assembly.FCStd`

An `Assembly::AssemblyObject` container Bradley started (empty as of 2026-09-07 — no bodies
linked in yet, no joints defined). Intended for physically verifying the two-tile +
connector fit (per `plan.md`'s validation section) — not yet populated.

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — ~22 variables | — | ✅ |
| `Base.FCStd` | One tile: base + stepped ramp (7 rows × 3 cols) + 4 connector pockets (2 pairs/side, 36 objects) | `Params.FCStd` | ✅ audit clean |
| `ConnectorStrip.FCStd` | Bridging connector, 2 pins | `Params.FCStd` | ✅ audit clean |
| `Assembly.FCStd` | Empty assembly container (WIP, not yet populated) | — | ✅ (trivially, nothing to break yet) |

No file is ❌ BROKEN.

---

## Params variables (summary)

`Params.FCStd` (`VarSet`), referenced as `<<Params>>#VarSet.VarName` (or `Params#VarSet.` in
same-workbench expressions):

| Group | Variables |
|---|---|
| Base geometry | `Width`(derived, 198.6mm) · `Depth`(150mm) · `LowerBackHeight`(20mm) · `BackHeight`(20mm) — both changed live by Bradley from the 15/30mm values noted in earlier commits; current values are authoritative |
| Shelf/slot | `ShelfDepth`(15mm) · `SlotDepth`(10mm) · `SlotHeight`(2mm) · `SlotWidth`(60.2mm) · `SlotSpacing`(3mm, shared use — hard rule 5) |
| Pattern | `NumColumns`(3, Integer) · `RowCount`(7, Integer) · `RowPitch`(35mm) |
| Connector (active) | `ConnectorPinSpan`(30mm) · `ConnectorRadius`(5mm) · `ConnectorPinHeight`(5mm) · `ConnectorBaseThickness`(1mm) · `ConnectorChamfer`(1mm) · `ConnectorClearance`(0.2mm) · `ConnectorInset`(15mm) · `ConnectorPairOffset`(50mm, added 2026-09-07 for the 2nd pair) |
| Snap-tab (parked, unused) | `SnapTabWidth`(14mm) · `SnapTabHeight`(20mm) · `SnapTabLength`(8mm) · `SnapTabClearance`(0.15mm) |

`Width = NumColumns * (SlotWidth + 2*SlotSpacing)` — **derived, not independent**. If you
ever need a truly independent tile width again (decoupled from the column math), that's a
deliberate redesign, not a quick edit — it was made derived specifically to close a real
drift bug (hard rule 1).

---

## How to verify your change didn't break parametric

```bash
python3 scripts/audit_parametric.py
```

Flags: unconstrained/underconstrained sketches, unbound dimensional constraints, sketches
attached to feature faces, unbound Pad/Pocket/Chamfer/Fillet numeric properties.

**What it does NOT catch** (see hard rule 1 for the full list, learned the hard way):
`AttachmentOffset` literals, pattern `Occurrences`/`Offset`, a VarSet property that's drifted
from a formula used elsewhere, and an `AttachmentSupport` that silently went empty. After any
attachment/pattern/Param change, manually check `ExpressionEngine` and cross-check related
Params — don't rely on the script alone. Also run `validate_document()`/`validate_object()`
after structural changes (ignore its sketch "non-positive volume" warning — false positive,
hard rule 9) and, for any new Pocket, verify by diffing `Shape.Volume` before/after against
the expected cut volume.

Baseline: **0 issues across all 4 files.**

**Documented script correction (this project's copy only):** the canonical script (and the
independently-corrected Clocks copy) still carry a DAG-risk regex bug — when a sketch's
`AttachmentSupport` is empty, the regex matches the *next* `<Link>` anywhere later in the
object. Fixed here by scoping the search to the `AttachmentSupport` property's own span. Not
propagated to canonical/Clocks — ask Bradley first. The `DIMENSIONAL_TYPES` enum fix is
already included (started from the Clocks-corrected version).

No exemptions beyond the standard ones (B-spline `Weight`, 90° `Angle`, inert `Length2`).

**Known-broken/unreliable typed MCP tools (2026-09-06/07, this FreeCAD 1.1.3 install):**
- `pad_sketch` throws on every call (`AttributeError: 'PartDesign.Feature' object has no
  attribute 'Symmetric'`). Use `execute_python` `doc.addObject("PartDesign::Pad", ...)`.
- `pocket_sketch`'s default `Reversed` direction isn't reliable — verify by volume diff.
- `linear_pattern` (typed) only supports axis-aligned `X`/`Y`/`Z` and one `feature_name` —
  can't do a diagonal direction or multiple `Originals`. Build via `execute_python`.
- `create_sketch` can silently create the sketch in the **wrong document** when multiple
  open documents have same-named bodies (both `Base` and `ConnectorStrip` have a body named
  `Body`) and the active document isn't the one you passed `doc_name` for — the tool
  returned success with no error, but the object appeared in neither document's tree.
  **Always explicitly target `doc = FreeCAD.getDocument("...")` via `execute_python`** when
  multiple documents are open, rather than trusting `doc_name` on typed tools.

---

## Memory files (deeper context)

No project-scoped memories yet.

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.**
- **MCP server auto-starts with FreeCAD.**
- **Write changes as `macros/*.FCMacro` files** — aspirational here; this project's work has
  been done interactively via `execute_python` (verify-each-step), matching how Bradley
  iterates directly in FreeCAD. Worth capturing the final state as a macro once it settles.
- **Cross-document expressions**: canonical form `<<Params>>#VarSet.VarName` (or
  `Params#VarSet.VarName` inside `execute_python`, which resolves the same way).
- **Run `python3 scripts/audit_parametric.py` before committing** — necessary, not sufficient.

**Project-specific:**

- `list_documents`'s `is_modified` flag is unreliable — don't trust it, just save.
- Multiple open documents with same-named bodies (`Base`/`ConnectorStrip` both have `Body`)
  — always target `execute_python` with an explicit `FreeCAD.getDocument(name)`, don't rely
  on typed-tool `doc_name` resolution alone.
- When a `PartDesign::Pocket`/`Pad` gives `Standard_NullObject NULL shape` after 1-2
  property changes, stop mutating it — delete and recreate fresh (hard rule 7).
- After creating any new tip feature via raw scripting, explicitly manage
  `ViewObject.Visibility` (hard rule 8) — nothing does it for you outside the GUI/typed tools.

---

## Print profile

**No successful test print yet — profile TBD.**
