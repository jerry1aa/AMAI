## Add Default Agent Workflow To Repo Instructions

### Summary
Document the default collaboration model for this repository so future tasks use the same structure:
- main agent handles research, planning, review, and integration
- one subagent may handle bounded coding and testing work when delegation is useful

### Implementation Changes
- In `AGENTS.md`:
  - add a repo-level workflow rule describing when to use the main agent and when to delegate to a subagent
- In `CHANGELOG.md`:
  - record the instruction update under `Unreleased`

### Test Plan
- Verify the new instruction is clear, project-scoped, and does not imply mandatory subagent use on every small task.
- Confirm the instruction preserves main-agent responsibility for repo-rule compliance and final review.
