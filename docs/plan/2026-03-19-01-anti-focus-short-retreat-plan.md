## Add Anti-Focus Short-Retreat Micro for Heroes and Fragile Units

### Summary
Add an early anti-focus reaction so heroes and fragile non-hero combat units briefly disengage when they take a burst of damage in a short time under unsafe local pressure. This is not full flee logic; it is a short retreat/backstep meant to break enemy focus fire before the unit reaches the normal rescue threshold.

### Implementation Changes
- Add shared anti-focus settings in `common.eai`:
  - hero burst-damage threshold
  - unit burst-damage threshold
  - anti-focus retreat distance
  - anti-focus cooldown duration
  - density disadvantage threshold
- Add a shared anti-focus cooldown state in `common.eai`:
  - units in anti-focus cooldown should temporarily reject focus-fire attack orders
- Add shared helpers in `common.eai`:
  - track recent non-hero HP loss with a lightweight hashtable
  - compute a short retreat point using allied/army fallback anchors and danger direction
  - issue a `move` order and queue guard reset similar to existing micro patterns
- Hero logic in `Jobs/MICRO_HERO.eai`:
  - use existing `hero_hp_loss[hn]`
  - before `SaveHero`, if burst HP loss is high and local density is unsafe:
    - trigger short anti-focus retreat
    - add hero to anti-focus cooldown
- Non-hero logic in `Jobs/MICRO_UNITS.eai`:
  - apply first version only to fragile non-heroes:
    - ranged attackers
    - mana/support/backline units
  - when recent HP loss is high and local pressure is unsafe:
    - trigger short anti-focus retreat
    - add unit to anti-focus cooldown
- Focus-fire integration in `Jobs/FOCUSFIRE_CONTROL.eai`:
  - exclude units in anti-focus cooldown from new focus-fire attack orders
- Keep existing rescue/flee systems intact:
  - `SaveHero`
  - `SaveUnit`
  - retreat control
  - hero chase safety
- Update `CHANGELOG.md`

### Test Plan
- Build all supported versions:
  - `MakeTFT.bat`
  - `MakeROC.bat`
  - `MakeREFORGED.bat`
- In-game checks:
  - hero under sudden focused damage briefly steps back before normal flee threshold
  - ranged/caster/support units under burst damage briefly disengage instead of standing still until near death
  - melee frontline does not overreact and cause army-wide stutter
  - anti-focus units are not immediately re-ordered into focus-fire during cooldown
  - full flee/rescue still triggers when HP continues dropping

### Assumptions
- Version 1 targets heroes and fragile non-hero backline units only.
- Trigger requires both burst HP loss and unsafe local pressure.
- Anti-focus response is a short move disengage, not a full send-home or healer action.
