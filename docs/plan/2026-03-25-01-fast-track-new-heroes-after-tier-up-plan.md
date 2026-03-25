# Fast-Track New Heroes After Town-Hall Tier Ups

## Summary
- Prioritize `hero[2]` as soon as tier-2 hero prerequisites are complete.
- Prioritize `hero[3]` as soon as tier-3 hero prerequisites are complete.
- Apply the change in shared `common.eai` logic so it covers TFT, ROC, REFORGED, all races.

## Key Changes
- Add shared priority floors:
  - `hero2_fast_track_prio = 130`
  - `hero3_fast_track_prio = 120`
- Add a shared helper that raises the effective queue priority for missing `hero[2]` / `hero[3]` only when their normal altar + hall prerequisites are already satisfied.
- Apply that boost inside the sorted build-queue insertion path so existing race strategy/build-sequence hero requests are accelerated without rewriting every race file.
- Leave hero identity selection, revive logic, and normal prerequisite checks unchanged.

## Test Plan
- Build `MakeTFT.bat`, `MakeROC.bat`, and `MakeREFORGED.bat`.
- Verify hero 2 is requested quickly after tier 2 finishes.
- Verify hero 3 is requested quickly after tier 3 finishes where the strategy includes a third hero.
- Verify no bypass of altar or hall prerequisites.
- Verify revive behavior still uses existing hero revive priorities.

## Assumptions
- Fast-tracking should apply to both hero 2 and hero 3.
- Shared queue-priority boosting is preferred over editing race build sequences individually.
