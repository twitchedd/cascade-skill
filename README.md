# Cascade agent skill

Teaches coding agents (Claude Code, Cursor, Codex, Copilot and any agent that reads `SKILL.md`)
to animate Unity **UI Toolkit** with [Cascade](https://u3d.as/45jT): tweens, springs, stagger,
variants, transitions and validated JSON animation sequences.

The skill detects whether your project has **Cascade Lite** (free, MIT, https://u3d.as/45jX) or
**Cascade Pro**, and only writes code your tier can compile.

## Install

```bash
npx skills add twitchedd/cascade-skill
```

Or copy `skills/cascade/` into your agent's skills folder (for Claude Code: `.claude/skills/cascade/`).

## Try it

> "Make the settings panel slide up and fade in when it opens."
>
> "Stagger the inventory slots in from the right."
>
> "Make this button pop when pressed."

## Contents

- `skills/cascade/SKILL.md`: tier detection, the fluent API, the rules that prevent common mistakes
- `skills/cascade/references/json-sequences.md`: the JSON sequence format (Pro)
- `skills/cascade/references/prompts.md`: intent → JSON examples
- `skills/cascade/references/cascade-sequence.schema.json`: machine-readable schema
