## Respect No Creep Limit Everywhere

### Goal
Make the commander `NO CREEP` restriction consistently prevent AI from attacking neutral creep camps.

### Findings
- Normal idle creeping is already blocked by `no_creep_attack`.
- Normal expansion creep-clearing is already blocked.
- Special creep paths still bypass the restriction:
  - militia expansion creep clearing
  - Ancient of War creep rush
  - ancient expansion creep clearing
  - setup jobs that start creep-only expansion flows

### Changes
1. Add a shared early helper/check for creep-attacking restrictions in `common.eai`.
2. Guard special creep attack paths with `no_creep_attack or ai_no_creep`.
3. Prevent ancient-expansion setup from starting when creep attacks are disabled.
4. Keep non-creep expansion logic unchanged.
5. Update `CHANGELOG.md`.

### Verification
- Run `MakeTFT.bat`, `MakeROC.bat`, `MakeREFORGED.bat`.
- Confirm parse success.
- Confirm no-creep mode skips special creep attack paths as well as normal idle creeping.
