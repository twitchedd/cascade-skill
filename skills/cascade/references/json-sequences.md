# Cascade JSON sequences (Pro)

Source: the Pro package's `Documentation~/AI-GUIDE.md`, sections 1–3. The installed package's own copy wins if they differ.

## 1. The JSON sequence format

```jsonc
{
  "steps": [                         // steps run ONE AFTER ANOTHER (sequential)
    {
      "actions": [                   // actions in a step run TOGETHER (parallel)
        { "verb": "SlideIn", "edge": "Bottom", "distance": 48, "duration": 0.4, "ease": "OutCubic" },
        { "verb": "Fade",    "to": 1, "duration": 0.4, "ease": "OutCubic" }
      ]
    },
    {
      "actions": [
        { "verb": "Scale", "to": 1.0, "duration": 0.3, "spring": "Bouncy" }
      ]
    }
  ]
}
```

**Mental model:** a step is a moment in time; everything in its `actions` plays at once. The next step
starts when the previous one finishes. This is the same as chaining fluent verbs before a `.Then()`.

### Per-verb fields

| `verb` | Target field(s) | Meaning | Cost |
|---|---|---|---|
| `Fade` | `to` (0–1) | opacity | cheap (transform) |
| `MoveX` | `to` (px) | translate X | cheap |
| `MoveY` | `to` (px) | translate Y | cheap |
| `MoveTo` | `x`, `y` (px) | translate to (x,y) | cheap |
| `Scale` | `to` (1 = 100%) | uniform scale | cheap |
| `Rotate` | `to` (degrees) | rotation | cheap |
| `SlideIn` | `edge`, `distance` (px) | enter from an edge toward rest | cheap |
| `BackgroundColor` | `color` | background color | cheap |
| `TextColor` | `color` | text color | cheap |
| `Nudge` | `x`, `y` (px) | displace and return — bumps, shakes, wobbles. **Additive:** composes with a `MoveX`/`MoveY` already animating the element instead of cancelling it | cheap |
| `Width` | `to` (px) | width | **reflows layout** |
| `Height` | `to` (px) | height | **reflows layout** |
| `Size` | `x`, `y` (px) | width, height | **reflows layout** |
| `Top` | `to` (px) | top offset | **reflows layout** |
| `Left` | `to` (px) | left offset | **reflows layout** |

Prefer the cheap transform verbs; only use layout verbs (Width/Height/Size/Top/Left) when you actually
need the box to resize/reposition.

### Shared fields (any verb)

| Field | Type | Default | Notes |
|---|---|---|---|
| `duration` | number (seconds) | `0.3` | clamped to ≥ 0 |
| `delay` | number (seconds) | `0` | wait before this action starts |
| `ease` | string | `OutCubic` | one of the easing names below |
| `spring` | string | — | if present, uses a physics spring and **ignores `ease`** |

### Vocabularies (use these exact names — case-insensitive)

- **ease:** `Linear`, `InSine`, `OutSine`, `InOutSine`, `InQuad`, `OutQuad`, `InOutQuad`, `InCubic`,
  `OutCubic`, `InOutCubic`, `OutQuart`, `OutExpo`, `OutCirc`, `InBack`, `OutBack`, `InOutBack`,
  `OutElastic`, `OutBounce`
- **spring:** `Gentle`, `Snappy`, `Bouncy`
- **edge:** `Left`, `Right`, `Top`, `Bottom`
- **color:** `"#RRGGBB"`, `"#RRGGBBAA"`, or a named color (e.g. `"red"`)

**Verb aliases (JSON ⇔ C#):** the verb name is the same in JSON and fluent C#, except the JSON parser also
accepts the C# method names as aliases — `"ScaleTo"` ⇔ `Scale`, `"FadeTo"` ⇔ `Fade`. So converting between
JSON and code (`element.Animate().Scale(...)` / `.FadeTo(...)` both exist) is mechanical. `MotionSettings.Describe()`
exposes each verb's `Aliases` and a `TryResolveVerb(name)` resolver if you need it programmatically.

A machine-readable JSON Schema is in `cascade-sequence.schema.json` next to this file — use it to
constrain or validate generated output.

---

### Where an action starts — read this before writing an entrance

Every verb animates **to** a value and reads its **from** off the element as it is at that moment. So:

```json
{ "verb": "Fade", "to": 1 }
```

on an element that is already visible animates 1 → 1. It parses, it validates, it runs, and **nothing
happens**. That is correct behaviour, and it is not what "fade in" means.

An entrance must say where it starts:

```json
{ "verb": "Fade", "from": 0, "to": 1, "duration": 0.3 }
```

`from` is optional and applies to the float-target verbs (`Fade`, `Scale`, `MoveX`, `MoveY`, `Rotate`,
`Width`, `Height`, `Top`, `Left`). Omit it whenever you mean "from wherever it is now" — which is right
for a hover, a settle, or the closing step of a shake. State it whenever the element has to *appear*.

`SlideIn` is the exception: it positions its own start from `edge` + `distance`, so it needs no `from`.

**Check yourself:** `SequencePreset.Lint(json)` returns an advisory for every action that would run and
change nothing, with the exact edit to make. `Validate` deliberately still passes such JSON — it answers
only "will this parse and run" — so lint as well as validate in a generate-and-fix loop.

## 2. Intent → JSON examples

**Fade in:**
```json
{ "steps": [ { "actions": [ { "verb": "Fade", "from": 0, "to": 1, "duration": 0.3, "ease": "OutCubic" } ] } ] }
```

**Slide up + fade in together (a classic entrance):**
```json
{ "steps": [ { "actions": [
  { "verb": "SlideIn", "edge": "Bottom", "distance": 40, "duration": 0.4, "ease": "OutCubic" },
  { "verb": "Fade", "from": 0, "to": 1, "duration": 0.4, "ease": "OutCubic" }
] } ] }
```

**Pop in** — a bouncy spring overshoots past the target on its own, so the overshoot is part of
the one action that travels, not a second step (a second `Scale to 1` would start from 1 and be inert):
```json
{ "steps": [ { "actions": [
  { "verb": "Fade",  "from": 0,   "to": 1, "duration": 0.15 },
  { "verb": "Scale", "from": 0.8, "to": 1, "spring": "Bouncy" }
] } ] }
```

**Attention shake (nudge right, then left, then settle):**
```json
{ "steps": [
  { "actions": [ { "verb": "Nudge", "x": 8, "y": 0, "duration": 0.22 } ] }
] }
```

**Notification breath (gentle pulse):**
```json
{ "steps": [
  { "actions": [ { "verb": "Scale", "to": 1.06, "duration": 0.5, "ease": "InOutSine" } ] },
  { "actions": [ { "verb": "Scale", "to": 1,    "duration": 0.5, "ease": "InOutSine" } ] }
] }
```

**Recolor on success:**
```json
{ "steps": [ { "actions": [ { "verb": "BackgroundColor", "color": "#2ECC71", "duration": 0.25, "ease": "OutCubic" } ] } ] }
```

---

## 3. How to apply a generated sequence (C#)

```csharp
using Cascade;

// A) Play immediately on an element (no asset needed):
SequencePreset.PlayJson(myElement, json);

// B) Validate first, then play — recover from a bad generation:
if (SequencePreset.TryFromJson(json, out var preset, out string error))
    preset.Play(myElement);
else
    Debug.LogWarning($"Could not parse animation: {error}");   // 'error' is specific; fix and retry

// B') In a self-correction loop, check validity without allocating an asset each try:
while (!SequencePreset.Validate(candidate, out string why))
    candidate = FixWith(why);   // 'why' is the same specific message as TryFromJson

// B'') STRUCTURED self-correction — branch on a code + path + suggestion instead of parsing a string:
MotionError err = SequencePreset.Validate(candidate);   // err.IsError == false when valid
if (err.IsError)
{
    // err.Code        -> MotionErrorCode.UnknownVerb / MissingField / NotFinite / OutOfRange / ...
    // err.Path        -> "steps[0].actions[1].to"   (the exact field to fix)
    // err.Suggestion  -> "Fade"   (nearest valid name for a typo'd verb/ease/spring/edge, else null)
    // err.Message     -> the same human string as the out-string overloads
    candidate = ApplyFix(candidate, err.Code, err.Path, err.Suggestion);
}

// C) Persist as a reusable asset and wire it codelessly to a UIAnimator rule
//    (designers then pick it in the inspector; see designer-workflow.md).
```

`TryFromJson` / `Validate(out string)` return a **specific** message (e.g. `step 0, action 1: verb 'Fade'
requires a numeric "to"`) — an AI loop should read it and correct the JSON rather than guess. For a
**programmatic** loop prefer `MotionError err = SequencePreset.Validate(json)`: branch on `err.Code`, jump
to `err.Path`, and apply `err.Suggestion` (a did-you-mean for typo'd names) — no string parsing. Use
`Validate` (not `TryFromJson`) when only checking, so you don't allocate a preset per attempt.

---

