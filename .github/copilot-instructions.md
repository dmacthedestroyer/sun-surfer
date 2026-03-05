# Sun Surfer — Development Instructions

## Spec-Driven Workflow

[SPEC.md](../SPEC.md) is the single source of truth for all game behavior, physics, visuals, and mechanics.

### Before making any code change:

1. **Read the spec.** Open [SPEC.md](../SPEC.md) before modifying `index.html`. Never rely on memory.
2. **Check alignment.** If the change contradicts the spec, stop and flag the conflict.
3. **Spec first, code second.** Behavior changes require a SPEC.md update _before_ or _alongside_ the code change. Code must never silently diverge from the spec.

### Spec edits:

- Describe _what_, not _how_. No implementation details.
- Precise, testable language (exact values, clear thresholds).
- Preserve section structure. Update the **Constants Reference** table when constants change.

### Code edits:

- Use named constants from the spec. No magic numbers.
- Match spec terminology ("flare" not "powerup", "burned" not "dead").
- Code/spec mismatch → flag it, don't silently reconcile.

## Project Structure

- `index.html` — Entire game (HTML + CSS + JS, single file, no dependencies)
- `SPEC.md` — Game specification (source of truth)
- `.github/copilot-instructions.md` — This file (workflow rules)
