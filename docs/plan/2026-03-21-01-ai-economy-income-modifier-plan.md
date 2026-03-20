## Add Configurable AI Economy Income Modifier

### Summary
Add a configurable way to strengthen or weaken AI gold and lumber income by percentage, while keeping the default behavior unchanged at `100%`.

### Implementation Changes
- In `common.eai`:
  - add global settings for `economy_gold_income_percent`, `economy_lumber_income_percent`, and `economy_income_interval`
- In `ROC/GlobalSettings.txt`, `TFT/GlobalSettings.txt`, and `REFORGED/GlobalSettings.txt`:
  - expose the new economy settings with neutral defaults
- In `Jobs.txt`:
  - register a new periodic `ECONOMY_INCOME` job that only runs when either percentage differs from `100`
- In `Jobs/ECONOMY_INCOME.eai`:
  - estimate active gold and lumber harvesting
  - apply runtime resource adjustments with `SetPlayerState(...)`
  - preserve fractional adjustments with remainders between job runs
  - respect upkeep on the gold-side estimate
- Update `CHANGELOG.md`

### Test Plan
- Build:
  - `MakeTFT.bat`
  - `MakeROC.bat`
  - `MakeREFORGED.bat`
- Verify:
  - `100%` settings produce no behavior change
  - values below `100` reduce AI effective resource gain over time
  - values above `100` increase AI effective resource gain over time
  - no new parse or compile errors in supported versions

### Notes
- Gold uses an upkeep-aware estimate based on active mine workers.
- Lumber uses a practical heuristic based on active lumber harvesters, so it is approximate rather than exact engine-level tracking.
