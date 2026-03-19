## Fix Anti-Focus Review Findings

### Summary
Address the three review findings from the first anti-focus implementation:
- preserve existing hero emergency flee priority
- keep non-hero HP-loss cache fresh outside battle mode
- remove the location leak in the danger-vector retreat fallback

### Implementation Changes
- In `Jobs/MICRO_HERO.eai`:
  - run the normal low-HP `SaveHero` path before anti-focus short retreat
- In `Jobs/MICRO_UNITS.eai`:
  - refresh recent HP-loss cache for eligible fragile units every micro cycle, not only while attacking/defending
  - still only trigger anti-focus retreat during `attack_running` or `town_threatened`
- In `common.eai`:
  - fix `GetAntiFocusRetreatLoc` cleanup so the danger-vector branch does not leak temporary locations
- Update `CHANGELOG.md`

### Test Plan
- Build:
  - `MakeTFT.bat`
  - `MakeROC.bat`
  - `MakeREFORGED.bat`
- Verify:
  - low-health heroes use `SaveHero` immediately even if anti-focus trigger also matches
  - previously damaged ranged/caster units do not falsely trigger anti-focus at battle start without fresh damage
  - no new parse errors

