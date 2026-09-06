# Project rules — Display Jewelry Cards

This is a countertop comb-style display rack for jewelry cards (earring/necklace cards):
a base block with a row of vertical slots, one card per slot. **The specific risk in this
project is incomplete parametrization of the ridge/slot feature** — `Pad.Length` (the base
block height) is a bare literal, and one dimension inside the slot-profile sketch
(`Sketch001.Constraints[4]`) is unbound. Both must get real Params variables before any
further slot-cutting work, per the global no-literals rule.

> **How to use this file:** every `[FILL: …]` marker from the bootstrap template has been
> filled from direct inspection (MCP + on-disk XML) as of 2026-09-06 — not guessed. Where
> something is inferred rather than confirmed, it's labeled below.

---

## Hard rules (this project)

These restate the global rules in `~/.claude/CLAUDE.md` with project-specific context.

1. **Everything parametric.** Worst offenders right now: `Pad.Length = 50mm` (literal, no
   Params var) and `Sketch001.Constraints[4]` (`DistanceY = 30mm`, literal). No prior
   coordinate-edit incident in this project — these are just gaps from work in progress, not
   damage. Fix by adding Params variables and binding via `setExpression`, never by typing
   the value back in.

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
- `Pad` extrudes `Sketch` 50mm in +Z — this is the base block. **`Length` is an unbound
  literal** (issue #1 above).
- `Sketch001` (`MapMode=ObjectXZ`, Body-local plane) defines a profile using `SlotHeight`
  (2mm), `SlotSpacing` (3mm), `SlotDepth` (25mm) as bound dimensions, plus one unbound
  `DistanceY=30mm` (issue #2 above) whose geometric role hasn't been traced yet.
- `Pad001` (`BaseFeature=Pad`, additive, direction `-X`, `Length` bound to `Width`) extrudes
  that profile across the full width, fusing onto `Pad`. This is a single ridge feature, not
  yet a repeated comb — **the actual row of card slots (presumably a `Pocket` + linear
  pattern) does not exist in the model yet.**

---

## Files in this project

| File | Role | Depends on | Status |
|---|---|---|---|
| `Params.FCStd` | VarSet — 5 variables (`Width`, `Depth`, `SlotDepth`, `SlotHeight`, `SlotSpacing`) | — | ✅ |
| `Base.FCStd` | Base block + first ridge feature (`Body`/`Sketch`/`Pad`/`Sketch001`/`Pad001`) | `Params.FCStd` | ⚠️ 2 unbound literals — see audit output below |

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

Baseline as of 2026-09-06 (after fixing a script bug — see below): **2 issues**, both real
(`Pad.Length` unbound, `Sketch001.Constraints[4]` unbound). Both are open work, not audit
noise — fix before the next geometry milestone.

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
