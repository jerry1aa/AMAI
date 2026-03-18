## Allow Upgrades During Town Threat

### Goal
Remove the global `town_threatened` gate from `StartUpgradeAM` so economic and combat upgrades can still be researched while the AI is defending.

### Changes
- Edit `common.eai`:
  - remove the early return in `StartUpgradeAM` that skips upgrades when `town_threatened` is true
- Update `CHANGELOG.md`

### Verification
- Run `MakeTFT.bat`
- Confirm no new parse errors
