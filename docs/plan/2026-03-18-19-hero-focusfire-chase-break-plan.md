## Hero Focus-Fire Chase Break Plan

### Goal
Reduce unsafe hero overchase after a focus-fire attack order has already been issued.

### Problem
Current logic has a strong pre-join filter in `HeroCanFocusFireTarget`, but once `FOCUSFIRE_CONTROL` issues a direct `attack` order, the hero can keep chasing a retreating target until the next focus-fire reevaluation tick.

### Design
1. Keep the existing join filter in `HeroCanFocusFireTarget`.
2. Add a second-stage chase-break helper in `Jobs/FOCUSFIRE_CONTROL.eai`, for example:
   - `HeroShouldContinueFocusFire takes unit hero, unit target returns boolean`
3. Use stricter continue conditions than join conditions. The helper should return `false` when any of these are true:
   - hero HP drops below the focus-fire HP threshold
   - local enemy density becomes too high relative to ally density
   - hero has moved too far from the supporting army center
   - target has moved too far from the supporting army center
   - hero-target distance becomes too large for a safe chase
4. Add a hero-specific filter step right before `GroupTargetOrder(..., "attack", target)`:
   - remove heroes that fail the continue check
   - leave non-hero units unchanged
5. If a hero is removed from focus-fire because the chase became unsafe:
   - do not issue the direct attack order to that hero
   - let normal army/captain control retake the hero on the next control cycle
6. Optional debug tag while tuning:
   - `FF: hero break chase`

### Suggested Continue Thresholds
- Hero HP: same as the current focus-fire join threshold
- Hero-to-army distance: tighter than join threshold
  - current join uses `normal_battle_radius * 0.60`
  - continue threshold can use about `0.45`
- Target-to-army distance: tighter than join threshold
  - current join uses `normal_battle_radius * 0.75`
  - continue threshold can use about `0.55`
- Hero-to-target chase leash:
  - add a new direct chase cap such as `normal_battle_radius * 0.45`
- Density:
  - keep current density rule or make continue rule slightly stricter

### Scope
- Main implementation in `Jobs/FOCUSFIRE_CONTROL.eai`
- No changes needed in `Jobs/MICRO_HERO.eai` unless later tuning shows the density inputs need to update faster

### Verification
- Run `MakeTFT.bat`
- Run `MakeROC.bat`
- Run `MakeREFORGED.bat`
- In-game test:
  - hero initially joins a normal nearby focus target
  - target retreats deep behind enemy lines
  - hero drops out of focus-fire and stops unsafe chasing earlier than before

### Expected Effect
- Heroes can still join nearby safe focus-fire
- Heroes stop participating in unsafe extended chase paths
- Normal units keep existing focus-fire behavior
