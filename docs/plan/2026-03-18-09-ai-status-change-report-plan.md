## Report all AI status changes to allies

- Replace the narrow attack-target notifier with a shared AI-command-state notifier based on `GetCurrentAICommandReport()`.
- Seed the cached report silently on the first periodic check so game start does not spam an initial idle message.
- Trigger immediate report checks from the high-level attack entry points so new attacks still announce without waiting for the periodic job.
- Trigger periodic report checks from `ChatVarsJob()` to catch status transitions like defend, retreat, and idle/regrouping.
- Update `CHANGELOG.md` and rebuild with `MakeTFT.bat`.
