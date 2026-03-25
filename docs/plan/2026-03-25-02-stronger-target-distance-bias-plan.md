# Stronger Target Distance Bias

## Summary
- Increase shared distance weighting so the AI more strongly prefers nearby valid player targets, creep camps, and expansion mines.
- Use the current main army position as the primary anchor, with major hero and home as fallbacks.
- Keep target safety and strength checks intact; this change biases choice toward local objectives instead of nearest-only behavior.

## Key Changes
- Add shared target-selection distance helpers and tuning constants in `common.eai`.
- Strengthen player-target distance scoring in `GetRangePenalty`, `GetWeakAndNearEnemy`, `GetSecondNearestEnemy`, and `GetTargetStrength` via the shared penalty.
- Rework creep selection in `GetFittingCreep` to score safe camps by strength difference plus distance instead of using only strength matching.
- Blend dynamic army/hero distance into expansion scoring so nearer valid expansions win more often without removing rebuild or previous-owner preference.

## Test Plan
- Run `MakeTFT.bat`, `MakeROC.bat`, and `MakeREFORGED.bat`.
- Verify attack target choice prefers a nearer valid enemy over a slightly weaker distant one.
- Verify creep choice favors a closer safe camp when multiple camps are similarly suitable.
- Verify expansion choice favors nearer valid mines more consistently, including rebuild candidates.

## Assumptions
- Distance bias is strong but not absolute nearest-only.
- Shared `common.eai` changes cover TFT, ROC, and REFORGED together.
