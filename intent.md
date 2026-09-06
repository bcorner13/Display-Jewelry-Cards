# Intent

## Goal

A countertop comb-style display rack for jewelry cards (earring/necklace cards): a solid
base block with a row of evenly-spaced vertical slots, one card standing upright per slot.

## Constraints

* Must follow `CAD_STANDARDS.md` (mm units, manifold geometry, centered on the print bed,
  $36–$45 price target, 2.0–3.0mm default wall thickness, Silk-filament-friendly aesthetic).
* Printable without supports.
* Countertop use — freestanding, not wall-mounted.

## Open questions (need Bradley's input before `plan.md` is finalized)

* **Card size**: what are the dimensions of the card each slot holds (width × thickness), and
  is it always one product line or does the rack need to handle a size range?
* **Capacity**: how many slots / cards should the rack hold? (Current `Width`=420mm suggests a
  wide rack, but that number may just be a placeholder from earlier work-in-progress.)
* **Slot pitch**: `SlotSpacing`=3mm and `SlotHeight`=2mm are already set in `Params.FCStd` —
  confirm these are real target values (card thickness + clearance) and not placeholders.
* Any branding/engraving on the base (e.g. a logo or shop name), per the Silk-filament
  aesthetic guidance in `CAD_STANDARDS.md`?
