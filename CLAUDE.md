# Project rules — Display Jewelry Cards

This is a countertop **tiered** display rack for 60×90mm earring cards standing upright
long-ways (90mm tall) on their short edge — **card dimensions are fixed**; row/column
counts, spacing, and the tile split are the adjustable levers, not the card size. Built as
**two identical tiles** (~200mm each — same part twice, not mirrored), for transport.
**Snap-tab interlock was tried and rejected 2026-09-06** ("won't hold them together") and
removed — deferred until columns/rows are settled; see Assembly architecture. As of
2026-09-06 the model has one continuous sloped shelf (`SideProfile`/`AngledTop`) with a
retention notch (`Shelf`/`Pocket`) and a card slot (`Slot`/`Pocket001`), patterned across
**3 columns** and **4 cascading rows** — `audit_parametric.py` clean.

> **How to use this file:** every `[FILL: …]` marker from the bootstrap template has been
> filled from direct inspection (MCP + on-disk XML) — not guessed. Where something is
> inferred rather than confirmed, it's labeled below.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** Currently clean (audit passes 0 issues), but this project has
   now hit the "unbound literal that isn't caught by the audit script" failure mode **twice**:
   - `Sketch003`("Shelf")'s `AttachmentSupport` had gone empty (see rule 3 below) — the
     audit script doesn't check for this, only for face-attachment.
   - `Sketch004`("Slot")'s `AttachmentOffset.Base.z` was a bare literal (`47`, approximating
     but not equal to the shelf's real flat-surface height `46.94`) — the audit script only
     checks Pad/Pocket/Chamfer/Fillet numeric properties and sketch *constraints*, **not**
     `AttachmentOffset` values. Fixed by binding it to the exact same expression that drives
     the shelf's flat step.
   - The column `LinearPattern`'s `Occurrences` was a plain `3` that happened to match the
     `NumColumns` Param — the Param existed but drove nothing. Fixed by binding it.
   - **Lesson: `audit_parametric.py` is necessary but not sufficient.** It doesn't check
     `AttachmentOffset` literals or `LinearPattern`/`PolarPattern` `Occurrences`/`Offset`
     bindings. When adding a new sketch attachment or pattern, manually verify every numeric
     input is expression-bound — don't rely on the script to catch it.
   - Earlier near-miss (pre-redesign): binding `Pad.Length` to the existing `Depth` variable
     instead of a new one — same numeric-literal fix, wrong concept, silently changed
     geometry as a side effect. Corrected to a dedicated variable. Same lesson: binding to
     *any* existing Param isn't enough — it has to be the right concept.

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this
   project; rule applies preventively. (The cautionary incident is the Spade Connector
   project, 2026-04-15/16 — documented globally.)

3. **Attach sketches to datum planes, not feature faces.** This project **hit exactly the
   failure mode this rule warns about**: an earlier `DatumPlane` was attached to
   `Pocket.Face12` (a feature face). When `Sketch003`'s geometry was rebuilt, that face
   reference broke (`validate_document` flagged `DatumPlane` invalid) — confirming
   face-attachment is fragile here, not just theoretically risky. The `DatumPlane` was
   unused by anything else and was deleted. Fix pattern used instead: attach to the Body's
   stable `ObjectXZ`/`YZ_Plane` datums, and if a sketch's geometry needs to track another
   feature's shape (e.g. the shelf notch tracking the ramp's slope), **compute the geometry
   parametrically from the same driving Params via expressions** — don't reference the
   feature's face at all. See Assembly architecture for the exact formula used.

4. **Clearance concepts stay decoupled.** `SlotWidth` (60.2mm = 60mm card + 0.2mm total
   clearance) is the card-slot print-fit dimension. `SnapTabClearance` (0.15mm/side,
   currently unused — see Assembly architecture) is a *different* mating interface. Don't
   merge them if/when the interlock work resumes.

5. **One knob, one concern — reused-but-legitimate case.** `SlotSpacing` (3mm) drives two
   *different* pieces of geometry (`SideProfile`'s short top-cap segment, and `Slot`'s two
   side-margin constraints) — this looks like the anti-pattern the rule warns about, but
   both uses represent the same real concept (a consistent small wall/margin thickness), so
   it's being treated as legitimate reuse rather than a collision. If the two ever need to
   diverge, split it into two Params then.

---

## Assembly architecture

Confirmed product intent (see `intent.md`): a countertop rack of 60×90mm earring cards,
standing upright, in a grid of **3 columns × 4 cascading rows** (12 cards, one tile).

Current geometry in `Base.FCStd` (one tile — 19 objects):

- `Body` → `Sketch` ("BoxFooter", `XY_Plane`, `Width`×`Depth` rectangle, 200mm×150mm) →
  `Pad` (base block, `Length`=`LowerBackHeight`=20mm).
- `Sketch001` ("SideProfile", `MapMode=ObjectXZ`, body-local datum — no `AttachmentSupport`
  needed, zero DAG risk) → `Pad001` ("AngledTop", `Length`=`Width`, additive, fused onto
  `Pad`). A 3-segment profile in the flat Y-Z plane (**not tilted** — the slope comes from
  the profile's own diagonal line, not a rotated sketch plane): front-bottom corner
  `(Depth/2, LowerBackHeight)` → back-top corner `(-(Depth/2-SlotSpacing), LowerBackHeight+
  BackHeight)` [the diagonal ramp] → back wall down to `(-(Depth/2-SlotSpacing), LowerBackHeight)`...
  actually the short cap closes at the *very* back edge; see live geometry for exact
  ordering. Bound to `BackHeight`/`SlotSpacing`.
- `Sketch003` ("Shelf", **rebuilt 2026-09-06** — see rule 3) → `Pocket` (the retention notch,
  `Length`=`Width`). Same flat `ObjectXZ` plane as `SideProfile` — **no face-attachment, no
  rotation**. A right-triangle wedge at the ramp's back-top corner:
  - `Yb = -(Depth/2 - SlotSpacing)` (ramp's back Y)
  - `Zt = LowerBackHeight + BackHeight` (ramp's top Z)
  - `h = ShelfDepth * BackHeight / (Depth - SlotSpacing)` (drop that lands back exactly on
    the ramp line — derived, not independently guessed; verified algebraically and by
    nudging `BackHeight` 30→45→30 and confirming the notch moved and returned correctly)
  - Corners: `(Yb, Zt)` → `(Yb, Zt-h)` → `(Yb+ShelfDepth, Zt-h)` → close (the closing edge
    coincides with the tail end of the ramp line itself).
- `Sketch004` ("Slot", `MapMode=FlatFace`, attached to the **datum** `XY_Plane` — proper,
  not a feature face) → `Pocket001` (the actual card-holding slot, `Length`=`SlotDepth`).
  Rectangular, `SlotWidth`(60.2mm)×`SlotHeight`(2mm), positioned at
  `AttachmentOffset.Base.z` = **the same expression as the Shelf's `h`-offset above**
  (fixed 2026-09-06 — was a bare literal `47` that merely approximated `46.94`).
- `LinearPattern` (columns): `Original`=`Pocket001`, `Direction`=`Sketch004.H_Axis`,
  `Mode=Spacing`, `Offset`=`SlotWidth + SlotSpacing*2`, `Occurrences`=`NumColumns` (3) —
  **`Occurrences` binding added 2026-09-06**; `NumColumns` existed but drove nothing before.
- `RowPattern` (rows — **added 2026-09-06**): `Original`=`Pocket001` (the **plain**, single
  slot feature — **not** the column `LinearPattern` object), `BaseFeature`=`LinearPattern`
  (chains onto the already-column-patterned tip), `Direction`=`(Sketch001, ['Edge1'])` (the
  ramp's own diagonal edge — repeats along the slope's exact current angle, whatever
  `BackHeight` is), `Mode=Spacing`, `Offset`=`RowPitch` (35mm, **provisional** — see below),
  `Occurrences`=`RowCount` (4).
  - **Important FreeCAD limitation discovered here:** a `PartDesign::LinearPattern` cannot
    take *another* `LinearPattern` object as its `Originals` — this produces a
    `Standard_NullObject … NULL shape` error on recompute, reproduced at trivial scale (2
    occurrences, 5mm offset), so it's structural, not a scale issue. **Workaround:** set
    `Originals` to the plain pre-pattern feature (`Pocket001`), and set `BaseFeature`
    explicitly to the already-patterned tip (`LinearPattern`) so the row-repeat fuses onto
    the column-patterned result instead of nesting a pattern inside a pattern's `Originals`.
    `PartDesign::MultiTransform` is the more "proper" FreeCAD mechanism for combining
    multiple transforms on one feature — not used here; this `BaseFeature`-chaining
    workaround is simpler and was verified to work (valid shape, correct volume, survives a
    `BackHeight` round-trip test).
  - **`RowPitch`=35mm is provisional**, chosen only to fit inside the ramp's actual usable
    length (~150mm; the ramp's own edge length is ~150.04mm, so 3 steps must total well
    under that — the first value tried, 60mm, overshot: 3×60=180mm > 150mm ramp length, and
    the row cuts would land off the physical ramp). This is exactly the **"tier offset" open
    question in `intent.md`** — not yet a final design decision, just a value that doesn't
    break geometry.

**Snap-tab interlock (added, then removed, 2026-09-06):** built a friction-fit peg+pocket
(`SnapTabSketch`/`SnapTabPad`/`SnapCatchSketch`/`SnapCatchPocket`), verified geometrically
correct (volume-diff matched to 3 decimals), but Bradley determined it **won't hold the two
tiles together** and deleted it. Deferred until after columns/rows are settled — a different
mechanism will be designed then. The `SnapTab*` Params (`SnapTabWidth`/`Height`/`Length`/
`Clearance`) remain in `Params.FCStd`, currently **unused** — not dead permanently, just
parked. Don't repurpose them for something else; the interlock work will resume.

(A "Sample Card" reference body — three 60×90mm rectangles at a rough, non-final pitch —
existed briefly on 2026-09-06 to confirm card orientation, then was deleted once confirmed.
Not present in the model; mentioned here only so a future session doesn't go looking for it.)

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — 14 variables | — | ✅ |
| `Base.FCStd` | One tile: base + ramp + shelf notch + card slot, patterned 3 cols × 4 rows (19 objects) | `Params.FCStd` | ✅ audit clean, `validate_document` all valid |

No file is ❌ BROKEN.

---

## Params variables (summary)

`Params.FCStd` (`VarSet`), referenced as `<<Params>>#VarSet.VarName`:

| Variable | Type | Value | Meaning |
|---|---|---|---|
| `Width` | Length | 200mm | This tile's width (X) |
| `Depth` | Length | 150mm | Base footprint depth (Y) |
| `LowerBackHeight` | Length | 20mm | Base block height (Z) before the ramp — changed from 50mm |
| `BackHeight` | Length | 30mm | Ramp's rise above the base |
| `ShelfDepth` | Length | 15mm | Retention-notch horizontal run |
| `SlotDepth` | Length | 10mm | Card slot cut depth — repurposed from the old retention-lip concept (was 25mm) |
| `SlotHeight` | Length | 2mm | Card slot opening height — repurposed (was the old lip notch height) |
| `SlotWidth` | Length | 60.2mm | Card slot width (60mm card + 0.2mm clearance) |
| `SlotSpacing` | Length | 3mm | Shared: `SideProfile`'s top-cap length AND `Slot`'s side margins (legitimate reuse, see hard rule 5) |
| `NumColumns` | Integer | 3 | Columns per tile — now actually wired to the column `LinearPattern` |
| `RowPitch` | Length | 35mm | Along-slope distance between rows — **provisional**, see Assembly architecture |
| `RowCount` | Integer | 4 | Rows (tiers) |
| `SnapTabWidth`/`Height`/`Length`/`Clearance` | Length | 14/20/8/0.15mm | Snap-tab interlock — **currently unused**, interlock removed pending redesign |

---

## How to verify your change didn't break parametric

After any FreeCAD edit, before considering the task done:

```bash
python3 scripts/audit_parametric.py
```

This script flags:
- Sketches with 0 constraints
- Sketches with dimensional constraints lacking expression bindings
- Sketches attached to feature faces (DAG risk)
- Feature dims (`Pad`/`Pocket`/`Chamfer`/`Fillet` `Length`/`Radius`/etc.) set as literals

**What it does NOT catch (confirmed the hard way this project, see hard rule 1):**
`AttachmentOffset` literals, `LinearPattern`/`PolarPattern` `Occurrences`/`Offset` bindings,
and a sketch whose `AttachmentSupport` silently went empty (frozen placement) rather than
pointing at a feature face. After adding/editing an attachment or a pattern, manually check
its numeric properties and `ExpressionEngine`, don't rely on the script alone.

Baseline: **0 issues.** Also run `validate_document()` after structural changes — it caught
the broken `DatumPlane` (attached to a feature face) that the audit script's regex missed
because nothing pointed the *sketch itself* there (only an orphaned datum did).

**Documented script correction (this project's copy only):** the canonical script (and the
independently-corrected Clocks copy) still carry a DAG-risk regex bug: when a sketch's
`AttachmentSupport` is empty, the regex scans past it (DOTALL) and matches the *next*
`<Link>` element anywhere later in the object. Fixed here by scoping the `<Link>` search to
the `AttachmentSupport` property's own span. Not yet propagated to the canonical or Clocks
copies — ask Bradley before doing so. The `DIMENSIONAL_TYPES` enum fix is already included.

No exemptions beyond the standard ones (B-spline `Weight`, 90° `Angle`, inert `Length2`).

**Additional known-broken typed MCP tools (2026-09-06, this FreeCAD 1.1.3 install):**
- `pad_sketch` throws `AttributeError: 'PartDesign.Feature' object has no attribute
  'Symmetric'` on every call — use `execute_python` `doc.addObject("PartDesign::Pad", ...)`
  instead (transaction rolls back cleanly on the failure, no orphaned object).
- `pocket_sketch`'s default cut direction isn't reliable — verify by diffing `Shape.Volume`
  before/after against the expected cut volume; don't trust the default `Reversed`.
- `linear_pattern` (typed tool) only supports axis-aligned `X`/`Y`/`Z` directions and a
  single `feature_name` — it cannot express a diagonal direction (needed for the row
  pattern) or multiple `Originals`. Built the row pattern via `execute_python` instead
  (`doc.addObject("PartDesign::LinearPattern", ...)`, set `Direction`/`Mode`/`Offset`/
  `Occurrences`/`Originals`/`BaseFeature` directly) — see Assembly architecture for the
  pattern-of-a-pattern workaround this also required.

---

## Memory files (deeper context)

No project-scoped memories yet. (Cross-project incident context: the Spade Connector
coordinate-edit saga in the global memory motivates hard rule #2.)

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not**
  `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use the FreeCAD Robust MCP server:
  `get_connection_status` first, then `open_document`, `list_objects`, `inspect_object`,
  `execute_python`, and macros. This is **enforced** by a PreToolUse hook in
  `.claude/settings.json` — raw shell access to `.FCStd` is blocked.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, the right
  interpretation is "FreeCAD isn't running" — ask whether to launch it.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits — noted as
  aspirational here; this session's fixes were done interactively via `execute_python`
  (verifying each step against the live model) rather than pre-written macros, matching how
  Bradley has been iterating directly in FreeCAD throughout. Worth writing a macro to
  capture the current final state for re-runnability once the design settles.
- **Cross-document expressions**: use the canonical form `<<Params>>#VarSet.VarName`.
- **Run `python3 scripts/audit_parametric.py` before committing** — and see the "what it
  does NOT catch" note above; it's necessary but not sufficient here.

**Project-specific:**

- `list_documents`'s `is_modified` flag has been observed to report `false` even when the
  live document differs from the saved file on disk. **Don't trust `is_modified` alone** —
  if in doubt, save.
- When rebuilding a sketch whose `AttachmentSupport` has gone empty (frozen placement,
  MapMode says `FlatFace` but nothing backs it), don't just re-attach to *a* face — check
  whether the geometry can instead be computed directly from the same Params that drive the
  feature it needs to track (see the Shelf/Slot fix above). This avoids the face-attachment
  fragility rule 3 warns about entirely, rather than just relocating it.
- A `PartDesign::LinearPattern` cannot pattern another `LinearPattern` (confirmed, not
  scale-dependent) — use the `Originals`=plain-feature + explicit `BaseFeature`=patterned-tip
  workaround, or `PartDesign::MultiTransform`.

---

## Print profile

**No successful test print yet — profile TBD.**
