# Hero Chase Safety Plan (2026-03-18)

## Goal
Reduce cases where AI heroes overchase retreating enemies into dangerous enemy territory during local battle targeting.

## Decisions
- Keep hero focus fire enabled, but make hero participation more conservative than regular combat units.
- Apply the new safety logic in local battle target selection (`FOCUSFIRE_CONTROL`) instead of removing heroes from attack systems globally.
- Use existing battle context where possible (`GetArmyOfUnit`, `army_loc`, `hero_enemy_density`, `hero_ally_density`) to avoid large architectural changes.

## Implementation
1. Add shared helper logic in `Jobs/MICRO_HERO.eai` or `common.eai` for hero chase safety:
   - detect whether a hero is too far from its army/battle center,
   - detect whether the hero is locally outnumbered,
   - return whether the hero may receive aggressive local focus-fire attack orders.
2. Update `Jobs/FOCUSFIRE_CONTROL.eai`:
   - when building the temporary focus-fire group, filter heroes through the new safety helper,
   - allow non-hero units to behave as before,
   - if a hero fails the safety check, exclude it from `GroupTargetOrder(..., "attack", target)`.
3. Keep the safety rule conservative:
   - hero can still attack local targets when near allied army support,
   - hero stops joining local chase if target pull distance becomes too large or local enemy pressure is too high.
4. Update `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat`.
- In-game checks:
  - hero still joins normal nearby fights,
  - hero stops following a retreating target when it pulls too far away from allied support,
  - ranged units still focus fire normally,
  - low-HP heroes still use existing save/teleport logic.

