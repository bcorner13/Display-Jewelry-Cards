# Project rules — Display Jewelry Cards

This is a countertop comb-style display rack for jewelry cards (earring/necklace cards):
a base block with a row of vertical slots, one card per slot. **As of 2026-09-06 the project
passes `audit_parametric.py` clean** — both prior gaps (`Sketch001.Constraints[4]` →
`BackHeight`, `Pad.Length` → `LowerBackHeight`) were fixed by Bradley directly in FreeCAD,
each with its own dedicated Params variable rather than reusing an existing one. Keep it
that way: the next real feature (the actual slot pattern) doesn't exist yet — see Assembly
architecture below.

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

**Inferred from the geometry, not yet confirmed against a written spec** (see `intent.md` /
`plan.md` for the confirmed product intent — countertop comb rack, per Bradley 2026-09-06):

- `Body` (label "Base") is a single `PartDesign::Body` in `Base.FCStd`.
- `Sketch` (on `XY_Plane`) is a `Width × Depth` rectangle (420mm × 150mm), fully constrained,
  both dimensions bound to Params.
- `Pad` extrudes `Sketch` 50mm in +Z — this is the base block. `Length` is bound to
  `LowerBackHeight` (its own dedicated Param, not reused from `Depth` or `BackHeight`).
- `Sketch001` (`MapMode=ObjectXZ`, Body-local plane) defines a profile using `SlotHeight`
  (2mm), `SlotSpacing` (3mm), `SlotDepth` (25mm), and `BackHeight` (30mm, added 2026-09-06)
  as bound dimensions — fully bound now.
- `Pad001` (`BaseFeature=Pad`, additive, direction `-X`, `Length` bound to `Width`) extrudes
  that profile across the full width, fusing onto `Pad`. This is a single ridge feature, not
  yet a repeated comb — **the actual row of card slots (presumably a `Pocket` + linear
  pattern) does not exist in the model yet.**

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — 7 variables (`Width`, `Depth`, `SlotDepth`, `SlotHeight`, `SlotSpacing`, `BackHeight`, `LowerBackHeight`) | — | ✅ |
| `Base.FCStd` | Base block + first ridge feature (`Body`/`Sketch`/`Pad`/`Sketch001`/`Pad001`) | `Params.FCStd` | ✅ fully bound, audit clean |

No file is ❌ BROKEN.

---

## Params variables (summary)

`Params.FCStd` (`VarSet`), all `App::PropertyLength`, referenced as `<<Params>>#VarSet.VarName`:

| Variable | Value | Meaning |
|---|---|---|
| `Width` | 420mm | Base block width (X) — also drives `Pad001.Length` |
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

Baseline as of 2026-09-06 (after fixing a script bug — see below): **0 issues.** Both
findings from earlier that day (`Sketch001.Constraints[4]`, `Pad.Length`) are now bound to
dedicated Params variables (`BackHeight`, `LowerBackHeight`). Keep it clean going forward —
run the audit after any geometry change, before considering it done.

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
