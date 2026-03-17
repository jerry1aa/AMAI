# Player Selection Control Handoff Plan (2026-03-18)

## Goal
Avoid AI-vs-player control conflict for AI units/heroes by pausing AMAI micro control when an allied human selects those units.

## Decisions
- Scope: micro/hero control only (no captain/commander attack flow changes).
- Trigger source: all allied human players.
- Resume behavior: AI control resumes 5 seconds after deselection.

## Implementation
1. Add shared lock helpers in `common.eai`:
   - detect allied-human selection of a unit,
   - store lock expiry time per-unit in `amaiCache`,
   - query whether a unit is AI-controllable.
2. Update `MICRO_UNITS`:
   - skip unit micro actions when unit is selected/locked.
3. Update `MICRO_HERO`:
   - skip hero micro tick when hero is selected/locked.
4. Update `CHANGELOG.md`.

## Validation
- Run `MakeTFT.bat` and `MakeOptTFT.bat`.
- Confirm parse/build success and no new errors.

