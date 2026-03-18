## Rich Economy Fast Tech Plan

### Goal
Make AI tier up to higher-tech halls faster when it has abundant resources, so it can access higher-level units earlier.

### Scope
- Cover all supported versions: `TFT`, `ROC`, and `REFORGED`.
- Apply to all four main races through shared logic where possible.
- Do not modify generated files under `Scripts/`.

### Design
1. Add a shared rich-economy helper in `common.eai`.
   - Detect when the AI is economically strong enough to accelerate hall tech.
   - Initial threshold proposal:
     - `GetGold() > 1400`
     - `GetWood() > 500`
2. Add a shared hall-tech priority helper in `common.eai`.
   - When the requested build target is the next racial hall tier, raise its effective priority under rich-economy conditions.
   - Keep the boost focused on tech halls only.
   - Initial priority target proposal:
     - tier 2 hall upgrade: `85`
     - tier 3 hall upgrade: `90`
3. Relax shared tier-up blockers under rich-economy conditions.
   - In the main build queue logic, when evaluating next-tier racial hall builds, reduce the effect of `bl_tier_unitlock` / related early-tier blocking if rich economy is active.
   - Keep expansion blocking and prerequisite checks intact unless they are clearly counterproductive.
4. Keep race build sequences unchanged unless necessary.
   - Prefer shared logic in `common.eai` so behavior stays consistent across races and versions.
5. Update `CHANGELOG.md`.

### Expected Effect
- AI retains normal early-game structure.
- Once floating substantial gold/wood, it prioritizes tier upgrades sooner.
- Higher-tier units and tier-gated upgrades become available earlier.

### Verification
1. Run:
   - `MakeTFT.bat`
   - `MakeROC.bat`
   - `MakeREFORGED.bat`
2. Confirm parse success for generated AI files.
3. Spot-check in code that racial hall upgrade requests now receive boosted priority under rich economy.
4. In-game expectation:
   - AI floating excess resources should advance hall tech sooner than before.
