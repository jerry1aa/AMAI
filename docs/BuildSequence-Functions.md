# BuildSequence.ai Function Reference

All functions available for use in `<VER>/<Race>/BuildSequence.ai`.

---

## Core Build Functions

| Function | Signature | Description |
|---|---|---|
| `BuildUnit` | `(qty, unitid, prio)` | Queue `qty` of a unit/building at priority `prio`. Most common function. |
| `BuildUpgr` | `(qty, unitid, prio)` | Queue an upgrade up to level `qty`. |
| `BuildItem` | `(qty, unitid, prio)` | Queue a purchasable item (neutral shop). |
| `BuildFront` | `(qty, unitid, prio)` | Build units at the **front** position (near battle). |
| `BuildAtMine` | `(qty, unitid, prio)` | Build units at the **expansion mine** location. |

---

## Advanced Build Functions

| Function | Signature | Description |
|---|---|---|
| `BuildAdvUpgr` | `(qty, upgid, starttier, unitcount, maxunits, tierprio, prio)` | Dynamic upgrade — priority scales up linearly as `unitcount` increases toward `maxunits`. |
| `BuildAdvUpgr2` | `(qty, upgid, starttier, unitcount, unitper, max, prio)` | Same idea but **random chance** — every `unitper` units gives +10% chance, capped at `max`%. |
| `SetBuildReact` | `(food, min1, unit1, min2, unit2, enemy_strength, strength1, strength2, prio)` | Build a reactive mix of two units based on detected **enemy strength**. Interweaves both unit types proportionally. |
| `BasicExpansionAM` | `(build_it, unitid, prio)` | Queue the next expansion hall if `build_it` is true and prerequisites are met. |

---

## Defend / Town Functions

| Function | Signature | Description |
|---|---|---|
| `DefendTowns` | `(qty, unitid, prio)` | Build `qty` units at each **expansion** town that has a mine. |
| `DefendTownsDone` | `(qty, unitid, prio)` | Same, but only for **completed** (fully built) expansion halls. |
| `DefendTownsFront` | `(qty, unitid, prio)` | Same as `DefendTowns` but **including main town**, built at front positions. |
| `DefendTownsFrontDone` | `(qty, unitid, prio)` | Same as `DefendTownsFront` but only completed halls. |
| `DefendTownsCond` | `(qty, unitid, min_dist, max_dist, min_gold, prio)` | Defend only towns that are far enough away and have enough gold remaining. |

---

## Dynamic Counter System (used together)

| Function | Signature | Description |
|---|---|---|
| `ResetDynamicSystem` | `()` | Clear all anti-X unit lists for this cycle. Call first. |
| `AddUnitToAnti<X>` | `(unitid, percent)` | Register `unitid` as a counter to enemy type X with a weighted `percent` chance. |
| `DynamicBuildUnit` | `(count, prio)` | Actually queue the chosen counter unit up to `count` food worth. |

Available `Anti<X>` types: `air`, `casters`, `heavyarmor`, `lightarmor`, `magic`, `mediumarmor`, `normal`, `piercing`, `siege`, `towers`, `unarmored`.

---

## Harass Functions (used in `init_strategy_*`)

| Function | Signature | Description |
|---|---|---|
| `AddHarass` | `(groupnum, qty, unitid)` | Define harass group `groupnum` to use `qty` of `unitid`. Called in `init_strategy`. |
| `Harass` | `(groupnum, target, avoid_towers, strength_limit, flee_percent, flee_number, cond, min_time, time)` | Trigger a harass attack for `groupnum` if conditions are met. |

---

## Flow Control Functions

| Function | Signature | Description |
|---|---|---|
| `SetTierBlock` | `(tier, unitper, foodlimit, expansionblock)` | Block tier-up until `unitper` fraction of requested units at `tier` are built and food < `foodlimit`. |
| `AddBlock` | `(req_qty, req_type, only_done, allow_qty, allow_type, expire_time)` | Add a rule: don't build more than `allow_qty` of `allow_type` until `req_qty` of `req_type` are built. |
| `basic_melee` | `(food, prio)` | Build a food-pool mix of the race's basic and advanced melee units (Undead-specific). |

---

## Notes

- **`prio`**: higher number = built sooner. Heroes typically at 80, key buildings 60–70, combat units 40–60, low-priority extras < 40.
- `unitid` always refers to the **AMAI internal constant** (e.g. `uHIPPO_RIDER`, `uANCIENT_WIND`), not the WC3 raw code.
- `BuildUnit`/`BuildUpgr` auto-queue **one level** of prerequisites via `RefreshNeeded` — but not recursively.
- All functions are defined in `common.eai` except `basic_melee` (defined in `races.eai`).
