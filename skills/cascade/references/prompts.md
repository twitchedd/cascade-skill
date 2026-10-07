# Cascade prompt pack — copy-paste intents for your AI assistant

Paste `AI-GUIDE.md` into your assistant once, then use prompts like these. Each entry shows the intent
you type and the JSON a well-guided assistant emits — play it with
`SequencePreset.PlayJson(element, json)`, or validate first with `SequencePreset.Validate(json, out var error)`.
(These mirror the intent chips in the demo scene — section 10.)

---

**"Make this panel pop in with a bouncy spring."**
```json
{ "steps": [ { "actions": [
  { "verb": "Scale", "from": 0.8, "to": 1, "spring": "Bouncy" },
  { "verb": "Fade", "from": 0, "to": 1, "duration": 0.3 }
] } ] }
```

**"Slide it up from the bottom and fade it in."**
```json
{ "steps": [ { "actions": [
  { "verb": "SlideIn", "edge": "Bottom", "distance": 48, "duration": 0.4, "ease": "OutCubic" },
  { "verb": "Fade", "from": 0, "to": 1, "duration": 0.4, "ease": "OutCubic" }
] } ] }
```

**"Shake it to say no."**
```json
{ "steps": [
  { "actions": [ { "verb": "MoveX", "to": -8, "duration": 0.05 } ] },
  { "actions": [ { "verb": "MoveX", "to": 8, "duration": 0.05 } ] },
  { "actions": [ { "verb": "MoveX", "to": -6, "duration": 0.05 } ] },
  { "actions": [ { "verb": "MoveX", "to": 0, "duration": 0.06 } ] }
] }
```

**"Dismiss it — fade out while shrinking slightly."**
```json
{ "steps": [ { "actions": [
  { "verb": "Fade", "to": 0, "duration": 0.3, "ease": "InCubic" },
  { "verb": "Scale", "to": 0.85, "duration": 0.3, "ease": "InCubic" }
] } ] }
```

**"Flash the background green to confirm, then settle back."**
```json
{ "steps": [
  { "actions": [ { "verb": "BackgroundColor", "color": "#2ECC71", "duration": 0.15 } ] },
  { "actions": [ { "verb": "BackgroundColor", "color": "#2D3138", "duration": 0.45, "ease": "OutCubic" } ] }
] }
```
*(Swap `#2D3138` for the element's resting color.)*

**"Pulse it gently so it draws attention."**
```json
{ "steps": [
  { "actions": [ { "verb": "Scale", "to": 1.08, "duration": 0.3, "ease": "InOutSine" } ] },
  { "actions": [ { "verb": "Scale", "to": 1, "duration": 0.35, "ease": "InOutSine" } ] }
] }
```

**"Grow the card to 200px wide, then snap its height taller with a spring."**
```json
{ "steps": [
  { "actions": [ { "verb": "Width", "to": 200, "duration": 0.35, "ease": "OutCubic" } ] },
  { "actions": [ { "verb": "Height", "to": 140, "spring": "Snappy" } ] }
] }
```
*(Width/Height reflow layout — fine for one-off emphasis, avoid per-frame use on big trees.)*

**"Land it level and settle with a spring."**
```json
{ "steps": [ { "actions": [
  { "verb": "Rotate", "from": -8, "to": 0, "spring": "Bouncy" },
  { "verb": "Scale", "from": 0.85, "to": 1, "spring": "Bouncy" },
  { "verb": "Fade", "from": 0, "to": 1, "duration": 0.25 }
] } ] }
```

---

## When the assistant gets it wrong

Feed the validator's message straight back to it — the errors are written to be actionable:

```
step 0, action 0: unknown MotionVerb 'Wobble' (one of: Fade, MoveX, MoveY, MoveTo, Scale, Rotate,
SlideIn, BackgroundColor, TextColor, Width, Height, Size, Top, Left)
```

```csharp
if (!SequencePreset.Validate(json, out var error))
    Debug.Log(error);            // paste this back to your assistant; it will correct the JSON
```

Tips that keep assistants on the rails: durations are **seconds** (0.3 = 300 ms); opacity is 0–1; scale
is a multiplier; verbs inside one step play together, steps play in order; `"spring"` overrides
`"ease"`. The machine-readable source of truth is `MotionSettings.Describe()` (every verb, field, vocabulary
and range) — an agent with code access can introspect it directly.
