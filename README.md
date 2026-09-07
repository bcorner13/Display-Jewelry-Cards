# Display Jewelry Cards

Parametric countertop tiered display rack for 60×90mm earring cards, standing upright. One
tile holds a 3-column × 7-row grid of card slots on real discrete stepped tiers (a
staircase). Two tiles (same part, printed twice) form the full display, joined underneath
by a separate connector strip.

## Files

| File | Description |
|------|-------------|
| `Params.FCStd` | VarSet — all parametric variables |
| `Base.FCStd` | One tile: base block + 7-step staircase + 21 card slots + 2 bottom connector pockets |
| `ConnectorStrip.FCStd` | Bridging connector — capsule with a pin at each end |
| `Assembly.FCStd` | Assembly container for fit-checking two tiles + connector (not yet populated) |

## Status

Columns × rows and the connector interlock are both modeled and working (audit-clean,
verified by volume math and screenshots). Still open: populate `Assembly.FCStd` to verify
physical fit, then a test print to tune fit tolerances. See `intent.md` for open questions
and `plan.md` for the feature tree.

## Parametric usage

1. Open `Params.FCStd`, `Base.FCStd`, and `ConnectorStrip.FCStd` together in FreeCAD 1.1+
   (`Base` and `ConnectorStrip` both cross-reference `Params` via
   `<<Params>>#VarSet.VarName` expressions).
2. Edit `VarSet` in `Params.FCStd` — e.g. `NumColumns`, `RowCount`, `RowPitch`.
3. Recompute.

Key parameters (`Params.FCStd`):
- `Width`/`Depth` — tile footprint, mm (`Width` is **derived** from `NumColumns`/
  `SlotWidth`/`SlotSpacing`, not independent — 198.6 × 150)
- `LowerBackHeight`/`BackHeight` — base block height / ramp rise, mm (15 / 30)
- `SlotWidth`/`SlotHeight`/`SlotDepth` — one card slot's dimensions, mm (60.2 / 2 / 10)
- `NumColumns`/`RowPitch`/`RowCount` — pattern controls (3 / 35 / 7)
- `Connector*` — bridging-connector geometry and print-fit clearance, shared between
  `Base.FCStd`'s bottom pockets and `ConnectorStrip.FCStd`'s pins
- `SnapTab*` — parked, unused (earlier interlock attempt, superseded by the connector strip)

## Before committing

```bash
python3 scripts/audit_parametric.py
```

Must be clean before any commit. **Note:** the audit script doesn't catch every parametric
gap in this project (bare `AttachmentOffset` literals, unwired pattern `Occurrences`, a
VarSet property drifting from a formula used elsewhere) — see `CLAUDE.md` hard rule 1 for
the full list and how to check manually.
