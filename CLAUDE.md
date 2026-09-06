# Project rules — Display Jewelry Cards

This is a countertop **tiered** display rack for 60×90mm earring cards standing upright
long-ways (90mm tall) on their short edge — **card dimensions are fixed**; row/column
counts, spacing, and the tile split are the adjustable levers, not the card size. Cards are
arranged in a 5-column × 4-row grid (cascading tiers, theater-seating style), built as
**two mirror-symmetric half-tiles** (~210mm each, split down the center column) rather than
one 420mm plate, for transport. See `intent.md`/`plan.md` for the full spec and open
questions (confirmed 2026-09-06). **`Base.FCStd` now models one tile** — `Width` is that
tile's width (210mm), not the full assembly's. The base block + one shelf ridge are fully
parametric (`audit_parametric.py` clean); the actual per-column card slots and the other 3
tiers don't exist yet — see Assembly architecture below.

> **How to use this file:** every `[FILL: …]` marker from the bootstrap template has been
> filled from direct inspection (MCP + on-disk XML) as of 2026-09-06 — not guessed. Where
> something is inferred rather than confirmed, it's labeled below.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** Currently clean (audit passes 0 issues). No prior
   coordinate-edit incident in this project. Worth noting as a near-miss: the first attempt
   at fixing `Pad.Length` bound it to the existing `Depth` variable (150mm) instead of a new
   one — same numeric-literal fix, wrong concept (Depth is the base rectangle's Y-footprint,
   not the block's Z-height), and it silently changed the base height 50mm→150mm as a side
   effect. Corrected to a dedicated `LowerBackHeight` variable. **Lesson: binding an unbound
   dimension to *any* existing Param isn't enough — it has to be the right concept**, per
   hard rule 5 below.

2. **No fixing geometry by editing raw sketch coordinates.** No prior incident in this
   project; rule applies preventively. (The cautionary incident is the Spade Connector
   project, 2026-04-15/16 — documented globally.)

3. **Attach sketches to datum planes, not feature faces.** No prior DAG incident in this
   project. `Sketch001` is attached via `MapMode="ObjectXZ"` (the Body's local XZ datum) with
   an **empty** `AttachmentSupport` — correctly datum-attached, not face-attached. It does use
   `Pad.Face3` as **external geometry** (a reference line for sizing the profile, not an
   attachment) — that's a softer dependency and not itself a DAG risk, but keep an eye on it
   if `Pad`'s face numbering ever changes.

4. **Clearance concepts stay decoupled.** No print-fit clearance Params exist yet — this
   project hasn't reached the fit-tolerance stage. When slot width becomes a real print-fit
   dimension (card thickness + slack), give it its own Param, e.g. `CardSlotClearance` — do
   not fold it into `SlotHeight` (that's the raw slot opening dimension, a different concept).

---

## Assembly architecture

Confirmed product intent (2026-09-06, see `intent.md`): a countertop rack holding 20
(5×4) earring cards, 60mm wide × 90mm tall, standing upright. 4 rows are tiered/cascading
(each set back + up from the one in front). The full 5-column row is built as 2
mirror-symmetric ~210mm tiles, split through the center column, to stay transportable.

Current geometry in `Base.FCStd` (one tile; the per-column slots and the other 3 tiers are
not modeled yet):

- `Body` (label "Base") is a single `PartDesign::Body`.
- `Sketch` (label "BoxFooter", on `XY_Plane`) is a `Width × Depth` rectangle (210mm × 150mm),
  fully constrained (incl. a `Symmetric` constraint about the Y-axis present since the
  start — the tile stays centered as `Width` changes), both dimensions bound to Params.
  `Width` was 420mm (full assembly) until 2026-09-06, when Bradley changed it to 210mm (one
  tile) as part of the two-tile split decision.
- `Pad` extrudes `Sketch` 50mm in +Z — the base block. `Length` is bound to
  `LowerBackHeight` (its own dedicated Param, not reused from `Depth` or `BackHeight`).
- `Sketch001` (label "SideProfile", `MapMode=ObjectXZ`, Body-local plane) defines the
  **row-1 (top row) retention-lip profile** in the Y-Z plane: a flat shelf floor at the top
  of `Pad` (Z=`LowerBackHeight`), an angled backrest rising toward the back, and a small
  lip near the back-top edge (`SlotHeight`×`SlotSpacing` notch, `SlotDepth`-long shelf run)
  that catches a card's top-back edge. Bound to `SlotHeight`/`SlotSpacing`/`SlotDepth`/
  `BackHeight`. This profile is a **cross-section**, not yet divided into per-card columns.
- `Pad001` (`BaseFeature=Pad`, additive, direction `-X`, `Length` bound to `Width`) extrudes
  that profile across the tile's full width, fusing onto `Pad`. **Single continuous ridge —
  no per-column dividers, no other 3 tiers, no interlock feature yet.**

(A "Sample Card" reference body — three 60×90mm rectangles at a rough, non-final pitch —
existed briefly on 2026-09-06 to confirm card orientation, then was deleted once confirmed.
Not present in the model; mentioned here only so a future session doesn't go looking for it.)

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — 7 variables (`Width`, `Depth`, `SlotDepth`, `SlotHeight`, `SlotSpacing`, `BackHeight`, `LowerBackHeight`) | — | ✅ |
| `Base.FCStd` | One tile: base block + row-1 shelf (`Body`/`Sketch`/`Pad`/`Sketch001`/`Pad001`) | `Params.FCStd` | ✅ fully bound, audit clean |

No file is ❌ BROKEN.

---

## Params variables (summary)

`Params.FCStd` (`VarSet`), all `App::PropertyLength`, referenced as `<<Params>>#VarSet.VarName`:

| Variable | Value | Meaning |
|---|---|---|
| `Width` | 210mm | **This tile's** width (X), not the full 5-column assembly's — changed from 420mm by Bradley 2026-09-06 as part of the two-tile split. Also drives `Pad001.Length` |
| `Depth` | 150mm | Base block depth (Y) |
| `SlotDepth` | 25mm | How far each card slot cuts in |
| `SlotHeight` | 2mm | Slot opening height (card thickness + clearance, currently undifferentiated — see hard rule 4) |
| `SlotSpacing` | 3mm | Center-to-center or wall spacing between slots |
| `BackHeight` | 30mm | Added 2026-09-06 by Bradley; binds `Sketch001.Constraints[4]` — a vertical offset in the ridge profile (exact geometric role still not traced in detail, but no longer an unbound literal) |
| `LowerBackHeight` | 50mm | Added 2026-09-06 by Bradley; binds `Pad.Length` — the base block's overall Z-height. Deliberately its own variable, not reused from `Depth` (footprint) or `BackHeight` (ridge-profile offset) |

Run `python3 scripts/audit_parametric.py --list-params` — **not yet implemented**; enumerate
manually via the table above or `execute_python` until that flag exists.

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

Baseline as of 2026-09-06 (after fixing a script bug — see below): **0 issues.** The two
original findings (`Sketch001.Constraints[4]`, `Pad.Length`) are bound to dedicated Params
variables (`BackHeight`, `LowerBackHeight`). A disposable "Sample Card" reference body
briefly reintroduced 12 findings and was deleted once it had served its purpose (confirming
card orientation) — see Assembly architecture.

**Documented script correction (this project's copy only):** the canonical script (and the
independently-corrected Clocks copy) still carry a DAG-risk regex bug: when a sketch's
`AttachmentSupport` is empty, the regex scans past it (DOTALL) and matches the *next*
`<Link>` element anywhere later in the object — which was this project's `Sketch001`
`ExternalGeometry` reference to `Pad.Face3`, producing a false "attached to feature face"
flag. Fixed here by scoping the `<Link>` search to the `AttachmentSupport` property's own
span. Not yet propagated to the canonical or Clocks copies — ask Bradley before doing so.

The `DIMENSIONAL_TYPES` enum fix (Radius/Diameter/Angle correctly mapped, Tangent/
Perpendicular/Block excluded) is already included — this project's copy started from the
corrected Clocks version, not the buggy canonical one.

No exemptions beyond the standard ones (B-spline `Weight`, 90° `Angle`, inert `Length2`).

---

## Memory files (deeper context)

No project-scoped memories yet — this is a fresh project. (Cross-project incident context:
the Spade Connector coordinate-edit saga in the global memory motivates hard rule #2.)

---

## Workflow notes

**Invariant (apply to every FreeCAD project — do not edit):**

- **Inspect/edit FreeCAD models via the MCP bridge — never with shell tools.** Do **not**
  `unzip`/`grep`/`cat`/`sed`/`strings`/etc. a `.FCStd`. Use the FreeCAD Robust MCP server:
  `get_connection_status` first, then `open_document`, `list_objects`, `inspect_object`,
  `execute_python`, and macros. This is **enforced** by a PreToolUse hook in
  `.claude/settings.json` — raw shell access to `.FCStd` is blocked. Only fall back to
  read-only `unzip` if the MCP bridge is genuinely unreachable, and ask the user first.
- **MCP server auto-starts with FreeCAD.** If an `mcp__freecad__*` call fails, the right
  interpretation is "FreeCAD isn't running" — ask whether to launch it. Do not silently fall
  back to `unzip` + XML parsing, and do not retry the same MCP call.
- **Write changes as `macros/*.FCMacro` files**, not direct XML edits.
- **Cross-document expressions**: use the canonical form `<<Params>>#VarSet.VarName`. The
  shorter `<<Params>>.VarName` form sometimes fails with "Params not found."
- **Run `python3 scripts/audit_parametric.py` before committing.**

**Project-specific:**

- `list_documents`'s `is_modified` flag has been observed to report `false` on this project's
  `Base` document even while the live document differed from the saved file on disk
  (confirmed 2026-09-06 by diffing on-disk XML against the live API — the live session had a
  Params binding and a datum reattachment that the last save didn't contain, yet
  `is_modified` said `false` both before and after that discrepancy existed). **Don't trust
  `is_modified` alone to decide whether a save is needed** — if in doubt, diff the on-disk
  XML against the live `ExpressionEngine`/`AttachmentSupport` state, or just save.
- `Sketch001` uses `Pad.Face3` as **external geometry** (not attachment) to size its profile
  against the base block. If `Pad`'s sketch or pad direction ever changes, re-verify this
  reference didn't silently repoint to the wrong face.

---

## Print profile

**No successful test print yet — profile TBD.**
