# Benefit Structure Simplification

## Goal

Reduce the visual length and repetition of the `Benefit Structure` capability while preserving the distinction between its three models.

## Approved design

- Rename the panel title to `Estructura del beneficio`.
- Keep the functional question and explanatory sentence unchanged.
- Remove the `Configuration & Conditions → Benefit Structure` distinction strip.
- Remove the repeated thesis above the model tabs.
- Remove the three-card comparison and final product implication.
- Keep the three model tabs.
- In `Multiple available`, render three independently activatable Coupons. Activating one must not disable the others.
- In `Choose one`, start without a preselected Coupon. Selecting one highlights it, disables the other two, and shows one compact selection result.
- Keep `Unlock & choose` unchanged.

## Interaction rules

- Each `Multiple available` Coupon changes from `Activar` to `Activo ✓` independently.
- `Choose one` permits one selected Coupon at a time and visually communicates mutual exclusivity.
- Existing model-tab behavior remains unchanged.

## Verification

- Confirm removed elements no longer render.
- Confirm all three `Multiple available` Coupons can be activated.
- Confirm `Choose one` starts unselected and locks competing options after selection.
- Confirm the panel has no horizontal overflow at desktop and mobile widths.
