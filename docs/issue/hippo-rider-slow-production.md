# Issue: Hippo Rider Production Is Broken / Combining Not Driven

## Affected Files

- `TFT/StandardUnits.txt` line 178
- `REFORGED/StandardUnits.txt` line 181
- `TFT/UnitEquivalence.txt` — HIPPO_RIDER missing
- `REFORGED/UnitEquivalence.txt` — HIPPO_RIDER missing
- `REFORGED/NeededExtra.txt` — HIPPO_RIDER missing
- `TFT/Elf/BuildSequence.ai` — Hippo2 tier 2 and tier 3

ROC is not affected (`needed1 = 0`, no combining needed).

---

## Background: How Unit Production Works in AMAI

When `BuildUnit(N, uHIPPO_RIDER, prio)` is called, `StartUnitAM` handles actual production.
It checks `UnitEquivalence.txt` for two special paths before falling through to the generic path:

| Path | Mechanism | Example |
|---|---|---|
| `UPGRADED_TO` | calls `SetProduce` on an upgraded unit type | Headhunter → Berserker |
| `BUILT_FROM` | calls `ConvertUnits` on an existing unit | Obsidian Statue → Black Sphinx |
| *(fallthrough)* | calls `SetProduce(qty, old_id[unitid], town)` | most regular units |

`SetProduce` is a WC3 native that trains a unit from a building.
`ConvertUnits` is a WC3 native that issues a transformation/ability order on existing units.

Hippogryph Rider (`ehpr`) is created in-game by an Archer using the "Mount Hippogryph" ability
on a nearby Hippogryph. It cannot be trained directly from a building. Therefore the correct
production path is `BUILT_FROM` → `ConvertUnits` on `ARCHER`.

---

## Root Cause 1 — `build_free` gate always blocks production (critical)

`StartUnitAM` uses `build_free[needed1[unitid]]` to track available factory slots:

```
if (buy_type[unitid] == BT_UNIT or ...) and CheckNotBuiltFrom(unitid) then
    set cost_qty = build_free[n1]      // n1 = needed1[unitid]
    set build_free[n1] = 0
else
    set cost_qty = need_qty
endif
set afford_qty = Min(cost_qty, afford_qty)
if afford_qty <= 0 then
    return NOT_ENOUGH_RES              // ← HIPPO_RIDER always hits this
endif
```

`build_free` is populated by `SetBuildFree`, which only iterates over the `building[]` array
(units with buy_type == BUILDING). `HIPPO` is a unit, not a building — it is never in
`building[]`, so `build_free[HIPPO]` is always 0.

With `needed1[HIPPO_RIDER] = HIPPO`:
- `n1 = HIPPO`
- `build_free[HIPPO] = 0` (never initialized)
- `cost_qty = 0` → `afford_qty = 0` → `NOT_ENOUGH_RES`

**`StartUnitAM` for HIPPO_RIDER returns `NOT_ENOUGH_RES` every single call. `SetProduce` is
never reached. Hippo Riders are completely blocked from production by the AMAI build system.**

The hippo riders occasionally seen in-game come from the WC3 engine's own fallback AI behaviour,
not from AMAI driving production.

When the `BUILT_FROM` fix is applied, `CheckNotBuiltFrom` returns false → the `build_free` block
is skipped → `cost_qty = need_qty` → production proceeds normally to `ConvertUnits`.

---

## Root Cause 2 — `BUILT_FROM` entry missing from UnitEquivalence.txt

`HIPPO_RIDER` is not listed in `TFT/UnitEquivalence.txt` or `REFORGED/UnitEquivalence.txt`.

Even if Root Cause 1 were fixed independently, `StartUnitAM` would fall through to
`SetProduce('ehpr', ...)` — a call that fails or is ignored since `ehpr` cannot be trained from
any building. The combining (mount) order is never issued by AMAI. Archers and Hippogryphs
pile up idle.

**Fix:** Add to `TFT/UnitEquivalence.txt` and `REFORGED/UnitEquivalence.txt`:
```
HIPPO_RIDER	BUILT_FROM	ARCHER	UPG_HIPPO_TAME	1
```
This makes `StartUnitAM` call `ConvertUnits(qty, old_id[ARCHER])` once `UPG_HIPPO_TAME >= 1`,
which issues the mount order on Archers so they combine with nearby Hippogryphs.

---

## Root Cause 3 — Wrong `needed1` in StandardUnits.txt (TFT and REFORGED)

The `needed1` column means **"the building that produces this unit"** — used by `RefreshNeeded`
to auto-queue factories at `prio + 6`. Compare:

```
HIPPO        → needed1 = ANCIENT_WIND   (correct: trains from AoW)
HIPPO_RIDER  → needed1 = HIPPO          (wrong: HIPPO is a unit, not a building)
```

`needed1 = HIPPO` causes `RefreshNeeded` to auto-queue HIPPO units at `prio + 6` (higher
priority than hippo riders) every time `BuildUnit(uHIPPO_RIDER)` is called. These hippogryphs
consume food and AoW slots.

Note: `SetBuildAllAMCore` does not read `needed1` — HIPPO_RIDER is still inserted into the
queue. The damage is done by `RefreshNeeded` running afterwards.

**Fix:** In `TFT/StandardUnits.txt` and `REFORGED/StandardUnits.txt`:

| Field   | Current (wrong) | Correct        |
|---------|-----------------|----------------|
| needed1 | `HIPPO`         | `ANCIENT_WIND` |
| needed2 | `ARCHER`        | `0`            |

Setting `needed1 = ANCIENT_WIND` also fixes the `build_free` gate independently (Root Cause 1):
`build_free[ANCIENT_WIND]` is a real building and is correctly initialized by `SetBuildFree`.

---

## Root Cause 4 — `REFORGED/NeededExtra.txt` missing HIPPO_RIDER

TFT and ROC both have this line in their `NeededExtra.txt`:
```
HIPPO_RIDER	UPGRADE	1	UPG_HIPPO_TAME
```
This blocks HIPPO_RIDER production until `UPG_HIPPO_TAME` is researched.

`REFORGED/NeededExtra.txt` does not have this line. Without it, the Reforged AI would attempt
to produce Hippogryph Riders before the Hippogryph Taming upgrade is done.

**Fix:** Add to `REFORGED/NeededExtra.txt`:
```
HIPPO_RIDER	UPGRADE	1	UPG_HIPPO_TAME
```

---

## Root Cause 5 — Explicit bare hippogryphs in Hippo2 build sequence

The Hippo2 tier 2 and tier 3 build sequences explicitly request bare hippogryphs:

```
// tier 2
call BuildUnit(10, uHIPPO_RIDER, 65)
call BuildUnit(4,  uHIPPO,       63)   ← wastes AoW slots and food

// tier 3
call BuildUnit(16, uHIPPO_RIDER, 65)
call BuildUnit(4,  uHIPPO,       63)   ← wastes AoW slots and food
```

With Root Cause 2 fixed, `BUILT_FROM` queues Archers for conversion. The AI needs some
Hippogryphs present for the mount to work, but those come naturally from `needed1 = ANCIENT_WIND`
auto-queuing. Explicit over-production of bare hippogryphs wastes food and AoW time.
These lines should be removed.

---

## Combined Effect

All root causes stack:
- `needed1 = HIPPO` → `build_free[HIPPO] = 0` always → HIPPO_RIDER **completely blocked** from AMAI production (Root Causes 1 + 3)
- No `BUILT_FROM` → mount order never issued even if blocking were lifted (Root Cause 2)
- Explicit `BuildUnit(4, uHIPPO)` wastes food/AoW on top (Root Cause 5)
- REFORGED missing `UPG_HIPPO_TAME` prerequisite check (Root Cause 4)

---

## Summary of Changes Needed

1. `TFT/UnitEquivalence.txt`: add `HIPPO_RIDER  BUILT_FROM  ARCHER  UPG_HIPPO_TAME  1`
2. `REFORGED/UnitEquivalence.txt`: same addition
3. `TFT/StandardUnits.txt`: HIPPO_RIDER — set `needed1 = ANCIENT_WIND`, `needed2 = 0`
4. `REFORGED/StandardUnits.txt`: same change
5. `REFORGED/NeededExtra.txt`: add `HIPPO_RIDER  UPGRADE  1  UPG_HIPPO_TAME`
6. `TFT/Elf/BuildSequence.ai`: remove `BuildUnit(4, uHIPPO, 63)` from Hippo2 tier 2 and tier 3
