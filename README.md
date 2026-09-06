# Display Jewelry Cards

Parametric countertop comb-style display rack for jewelry cards (earring/necklace cards) —
a base block with a row of vertical slots, one card per slot.

## Files

| File | Description |
|------|-------------|
| `Params.FCStd` | VarSet — all parametric variables |
| `Base.FCStd` | Base block + ridge feature (work in progress — slot cutting not yet modeled) |

## Status

Early parametric work-in-progress. See `intent.md` for the open product questions and
`plan.md` for the planned feature tree — the actual comb/slot pattern (the defining feature
of the product) has not been modeled yet, only the base block and one ridge feature.

## Parametric usage

1. Open `Params.FCStd` and `Base.FCStd` together in FreeCAD 1.1+ (both stay open — `Base`
   cross-references `Params` via `<<Params>>#VarSet.VarName` expressions).
2. Edit `VarSet` in `Params.FCStd` — e.g. `Width`, `Depth`, `SlotSpacing`.
3. Recompute `Base.FCStd`.

Current parameters (`Params.FCStd`):
- `Width` — base block width, mm (currently 420)
- `Depth` — base block depth, mm (currently 150)
- `SlotDepth` — how far each slot cuts in, mm (currently 25)
- `SlotHeight` — slot opening height, mm (currently 2)
- `SlotSpacing` — spacing between slots, mm (currently 3)

## Before committing

```bash
python3 scripts/audit_parametric.py
```

Must be clean (or have only documented exemptions — see `CLAUDE.md`) before any commit.
