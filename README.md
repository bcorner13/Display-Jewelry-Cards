# Display Jewelry Cards

Parametric countertop tiered display rack for 60×90mm earring cards, standing upright.
One tile holds a 3-column × 4-row grid of card slots on a single continuous sloped shelf.
Two tiles (same part, printed twice) form the full display; interlock mechanism TBD.

## Files

| File | Description |
|------|-------------|
| `Params.FCStd` | VarSet — all parametric variables |
| `Base.FCStd` | One tile: base block + ramp + retention shelf + card slot, patterned 3×4 |

## Status

Columns × rows are modeled and working (audit-clean, `validate_document` healthy). Interlock
between the two tiles is **open** — a snap-tab friction fit was tried and rejected (didn't
hold the tiles together) and removed; needs a real mechanical mechanism. See `intent.md` for
open questions and `plan.md` for the feature tree / what's been tried.

## Parametric usage

1. Open `Params.FCStd` and `Base.FCStd` together in FreeCAD 1.1+ (both stay open — `Base`
   cross-references `Params` via `<<Params>>#VarSet.VarName` expressions).
2. Edit `VarSet` in `Params.FCStd` — e.g. `NumColumns`, `RowCount`, `RowPitch`.
3. Recompute `Base.FCStd`.

Key parameters (`Params.FCStd`):
- `Width`/`Depth` — tile footprint, mm (200 × 150)
- `LowerBackHeight`/`BackHeight` — base block height / ramp rise, mm (20 / 30)
- `SlotWidth`/`SlotHeight`/`SlotDepth` — one card slot's dimensions, mm (60.2 / 2 / 10)
- `SlotSpacing`/`ShelfDepth` — shared margin / retention-notch run, mm (3 / 15)
- `NumColumns`/`RowPitch`/`RowCount` — pattern controls (3 / 35 / 4)
- `SnapTab*` — parked, unused (interlock removed pending redesign)

## Before committing

```bash
python3 scripts/audit_parametric.py
```

Must be clean (or have only documented exemptions — see `CLAUDE.md`) before any commit.
**Note:** the audit script doesn't catch every parametric gap in this project (bare
`AttachmentOffset` literals, unwired pattern `Occurrences`) — see `CLAUDE.md` hard rule 1.
