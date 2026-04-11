# Unit Combination Paths: MergeUnits vs ConvertUnits

## Summary

There are two mechanisms for unit combination (e.g. ARCHER + HIPPO → HIPPO_RIDER):

- **`ConvertUnits`** — single-source only (e.g. OBSIDIAN_STATUE → BLK_SPHINX). Driven by `BUILT_FROM` entries in `UnitEquivalence.txt`.
- **`MergeUnits`** — two-source (e.g. ARCHER + HIPPO → HIPPO_RIDER). Driven by `UnitConversions.txt`. Called in **both** the build phase and the attack phase.

## The Two Paths

### Path 1 — ConvertUnits (build phase, via `RefreshNeeded` in `common.eai`)

- Triggered by `BuildUnit(uBLK_SPHINX)` when a `BUILT_FROM` entry exists in `UnitEquivalence.txt`.
- Calls: `ConvertUnits(afford_qty + have_qty, oOBSIDIAN_STATUE)` — one source unit.
- **Native signature**: `native ConvertUnits takes integer qty, integer id returns boolean`
- **Only works for single-source transformations.** Cannot handle two-source merges.

### Path 2 — MergeUnits (build phase, via `RefreshNeeded` in `common.eai`)

- Triggered by `BuildUnit(uHIPPO_RIDER)` when a `UnitConversions.txt` entry exists.
- Entry format: `Name in AI | Merge from 1 | Merge from 2 | Upgrade ID | Upgrade Level`
- Calls: `MergeUnits(afford_qty + have_qty, oARCHER, oHIPPO, oHIPPO_RIDER)`
- **Native signature**: `native MergeUnits takes integer qty, integer a, integer b, integer make returns boolean`
- Fires every build cycle as soon as the upgrade gate is met and source units are available.

### Path 3 — MergeUnits (attack phase, via `setup_force` → `ConversionsAM` in `common.eai`)

- Triggered once per attack wave inside `setup_force()` in `races.eai`.
- Call chain: `setup_force()` → `SetMeleeGroupAM()` → `SetAssaultGroupAM(0, 60, unitid)` → `ConversionsAM(60, unitid)` → `MergeUnits(desire, oARCHER, oHIPPO, oHIPPO_RIDER)`
- Also reads `UnitConversions.txt`. Uses only columns %1–%3 (no upgrade gate check here).
- Merges whatever source units remain after the build phase already combined what it could.

## Experimental Confirmation (2026-04-10)

Verified by commenting out each call independently in `common.eai`:

| Action | Result |
|---|---|
| Comment out `MergeUnits` at `common.eai:13073` (attack phase) | No archer+hippo → hippo_rider combination occurs |
| Comment out `ConvertUnits` at `common.eai:12017` (build phase) | Combination still occurs normally |

This confirmed that at that time, the attack-phase `MergeUnits` was the only effective path. The build-phase `MergeUnits` block was subsequently added (2026-04-11) to also merge during peacetime.

## Data File Roles

| File | Used by | Purpose |
|---|---|---|
| `UnitEquivalence.txt` (BUILT_FROM) | `RefreshNeeded`, `CheckNotBuiltFrom` | Single-source conversions + gating from `SetProduce` |
| `UnitConversions.txt` | `RefreshNeeded`, `ConversionsAM`, `CheckNotBuiltFrom` | Two-source merge definitions + upgrade gate |

## Key Rules

- **Two-source units** (HIPPO_RIDER): use `UnitConversions.txt`. Do NOT add a BUILT_FROM entry in `UnitEquivalence.txt` — the ConvertUnits path cannot handle two sources and would return `CANNOT_BUILD` early, blocking the MergeUnits block.
- **Single-source units** (BLK_SPHINX): use `BUILT_FROM` in `UnitEquivalence.txt`. No entry in `UnitConversions.txt` needed.
- Both `UnitConversions.txt` and `UnitEquivalence.txt` BUILT_FROM gate `CheckNotBuiltFrom`, which prevents these units from going through the normal `SetProduce` path.

## Relevant Code Locations

- `common.eai:11678` — `CheckNotBuiltFrom`: excludes BUILT_FROM and UnitConversions units from normal production
- `common.eai:12018` — BUILT_FROM branch in `RefreshNeeded`, calls `ConvertUnits` (single-source)
- `common.eai:12037` — UnitConversions branch in `RefreshNeeded`, calls `MergeUnits` (two-source, build phase)
- `common.eai:13071` — `ConversionsAM` function, calls `MergeUnits` (two-source, attack phase)
- `races.eai:458` — `setup_force()`, calls `SetMeleeGroupAM` which triggers `ConversionsAM`
- `TFT/UnitEquivalence.txt` — BUILT_FROM entries (BLK_SPHINX only; HIPPO_RIDER removed)
- `TFT/UnitConversions.txt` — two-source merge definitions with upgrade gate columns
