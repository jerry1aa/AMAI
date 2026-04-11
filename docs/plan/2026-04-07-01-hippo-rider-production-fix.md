# Plan: Fix Hippo Rider Production (TFT + REFORGED)

Date: 2026-04-07
Issue: docs/issue/hippo-rider-slow-production.md

---

## Problem Summary

Hippogryph Rider production is completely broken in AMAI for TFT and REFORGED.
Five root causes were identified (see issue doc). The net effect:
- `StartUnitAM` always returns `NOT_ENOUGH_RES` for HIPPO_RIDER — AMAI never drives production
- The mount (Archer + Hippogryph → Hippogryph Rider) is never issued by AMAI
- Bare hippogryphs are auto-queued at high priority and over-produced, wasting food and AoW slots
- REFORGED is also missing the UPG_HIPPO_TAME prerequisite guard

---

## Changes

### Step 1 — Fix `needed1` and `needed2` in StandardUnits.txt (TFT + REFORGED)

**File:** `TFT/StandardUnits.txt` line 178
**File:** `REFORGED/StandardUnits.txt` line 181

Change HIPPO_RIDER:
- `needed1`: `HIPPO` → `ANCIENT_WIND`
- `needed2`: `ARCHER` → `0`

**Why:** `needed1` is the "factory building" field. Setting it to HIPPO (a unit) causes:
(a) `RefreshNeeded` to auto-queue hippogryphs at prio+6
(b) `build_free[HIPPO]` to be used as the factory slot counter — always 0 since only
    buildings are initialized in `SetBuildFree` — blocking production completely.

Setting `needed1 = ANCIENT_WIND` fixes both: correct factory tracking and no unwanted hippogryph flooding.
`needed2 = ARCHER` is also wrong (auto-queues archers unnecessarily); set to 0.

ROC already has `needed1 = 0, needed2 = 0` — no change needed there.

---

### Step 2 — Add `BUILT_FROM` to UnitEquivalence.txt (TFT + REFORGED)

**File:** `TFT/UnitEquivalence.txt`
**File:** `REFORGED/UnitEquivalence.txt`

Add line (tab-separated, after the existing `BLK_SPHINX` line for consistency):
```
HIPPO_RIDER	BUILT_FROM	ARCHER	UPG_HIPPO_TAME	1
```

**Why:** HIPPO_RIDER cannot be trained from a building — it is created by an Archer mounting
a Hippogryph (`ConvertUnits` native). Without this entry, `StartUnitAM` falls through to
`SetProduce('ehpr')` which fails silently. With this entry:
- `CheckNotBuiltFrom(HIPPO_RIDER)` returns false → `build_free` gate is bypassed
- `ConvertUnits(qty, old_id[ARCHER])` is called when `UPG_HIPPO_TAME >= 1` → mount order issued

---

### Step 3 — Add `UPG_HIPPO_TAME` prerequisite to REFORGED/NeededExtra.txt

**File:** `REFORGED/NeededExtra.txt`

Add line (tab-separated):
```
HIPPO_RIDER	UPGRADE	1	UPG_HIPPO_TAME
```

**Why:** TFT and ROC both have this guard. REFORGED is missing it, which would allow the AI
to attempt production before the Hippogryph Taming upgrade is researched.

---

### Step 4 — Rebalance hippogryph and archer counts in Hippo2 build sequence

**File:** `TFT/Elf/BuildSequence.ai`

`BuildUnit(uHIPPO)` and `BuildUnit(uARCHER)` must both stay in the sequence — they are the
raw inputs for the mount. `ConvertUnits(qty, old_id[ARCHER])` (issued by `BUILT_FROM`) pairs
an archer with a nearby hippogryph; if no hippogryphs exist, it returns `CANNOT_BUILD`.

The problem is the quantities: tier 2 targets 10 riders but only queues 4 hippos and 4 archers.
Each rider consumes 1 hippogryph + 1 archer, so the feeder counts should match the rider goal.
Additionally `uHIPPO_RIDER` (65) is higher priority than `uHIPPO` (63) and `uARCHER` (60),
meaning the AI attempts to mount before enough feeders are built.

**Changes for tier 2:**
```
// before
call BuildUnit(10, uHIPPO_RIDER, 65)
call BuildUnit(4,  uHIPPO,       63)
call BuildUnit(4,  uARCHER,      60)

// after
call BuildUnit(10, uHIPPO,       65)
call BuildUnit(10, uARCHER,      65)
call BuildUnit(10, uHIPPO_RIDER, 63)
```

**Changes for tier 3:**
```
// before
call BuildUnit(16, uHIPPO_RIDER, 65)
call BuildUnit(4,  uHIPPO,       63)
call BuildUnit(4,  uARCHER,      60)

// after
call BuildUnit(16, uHIPPO,       65)
call BuildUnit(16, uARCHER,      65)
call BuildUnit(16, uHIPPO_RIDER, 63)
```

**Why:** Build hippogryphs and archers first (higher priority), then `ConvertUnits` mounts them
into riders (lower priority, so it only fires once feeders are ready). Equal quantities ensure
no wasted hippogryphs or archers sitting idle.

---

## Build & Verify

After all edits:

1. Run `.\MakeTFT.bat` and `.\MakeREFORGED.bat` to recompile.
2. Verify `Scripts/TFT/` and `Scripts/REFORGED/` contain updated output.
3. Test in-game with Hippo2 strategy (Night Elf, TFT):
   - Confirm Archers are mounting Hippogryphs once UPG_HIPPO_TAME is done
   - Confirm no bare hippogryphs accumulating idle
   - Confirm army reaches tier 3 with a proper mass of Hippogryph Riders

---

## Order of Steps

Steps 1–4 are independent edits to different files; all can be done in any order before
rebuilding. Rebuild and test only once after all edits are complete.

---

## Summary of Changes (7 edits across 6 files)

1. `TFT/UnitEquivalence.txt` — add `HIPPO_RIDER  BUILT_FROM  ARCHER  UPG_HIPPO_TAME  1`
2. `REFORGED/UnitEquivalence.txt` — same addition
3. `TFT/StandardUnits.txt` — HIPPO_RIDER: `needed1 = ANCIENT_WIND`, `needed2 = 0`
4. `REFORGED/StandardUnits.txt` — same change
5. `REFORGED/NeededExtra.txt` — add `HIPPO_RIDER  UPGRADE  1  UPG_HIPPO_TAME`
6. `TFT/Elf/BuildSequence.ai` Hippo2 tier 2 — reorder: hippo(10,65) / archer(10,65) / rider(10,63)
7. `TFT/Elf/BuildSequence.ai` Hippo2 tier 3 — reorder: hippo(16,65) / archer(16,65) / rider(16,63)

---

## CHANGELOG Entry

```
- Fix Hippogryph Rider production broken in TFT and REFORGED:
  - Add HIPPO_RIDER BUILT_FROM ARCHER to UnitEquivalence.txt (TFT + REFORGED)
  - Fix HIPPO_RIDER needed1/needed2 to ANCIENT_WIND/0 in StandardUnits.txt (TFT + REFORGED)
  - Add UPG_HIPPO_TAME prerequisite for HIPPO_RIDER in REFORGED/NeededExtra.txt
  - Rebalance Hippo2 build sequence: equal hippo/archer/rider counts, feeders built first
```
